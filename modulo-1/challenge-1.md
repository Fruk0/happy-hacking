# Challenge — Módulo 1
## Operación Silent Recon

---

## El objetivo

Tu misión es construir el perfil de inteligencia más completo posible sobre la infraestructura pública de **Rappi** usando exclusivamente las herramientas y técnicas del módulo.

Al terminar tenés que poder responder una sola pregunta:

> *¿Qué tiene expuesto Rappi y por dónde entraría un atacante?*

**Target:** `rappi.com`

---

## Reglas

> [!IMPORTANT]
> **No enviar tráfico al objetivo.**
> Todo lo que hagas tiene que poder hacerse sin que Rappi lo detecte.
> Consultás fuentes públicas — no tocás sus servidores.

- Todas las técnicas son pasivas
- Guardá el output de cada herramienta en archivos desde el primer comando
- Cada hallazgo que vale la pena → captura de pantalla en el momento

---

## Tareas

### 🔵 Nivel 1 — Reconocimiento base

- [ ] Ejecutar `whois rappi.com` y anotar: registrar, nameservers, y fecha de creación
- [ ] Obtener todos los registros DNS (A, MX, NS, TXT) con `dig`
- [ ] Identificar el proveedor de correo a partir del registro MX
- [ ] Leer los registros TXT y listar todos los servicios que podés identificar
- [ ] Intentar zone transfer contra los nameservers encontrados — documentar el resultado exacto

### 🟡 Nivel 2 — Superficie de ataque

- [ ] Enumerar subdominios con `subfinder` y guardar el output
- [ ] Resolver los subdominios encontrados con `dnsx` — ¿cuántos están activos?
- [ ] Filtrar los subdominios más interesantes (`api.`, `dev.`, `admin.`, `staging.`, etc.)
- [ ] Elegir los **3 más interesantes** y justificar por qué llamaron la atención
- [ ] Buscar `org:"Rappi"` en Shodan y documentar al menos 2 hallazgos concretos

### 🔴 Nivel 3 — OSINT

- [ ] Ejecutar al menos **2 Google Dorks** con resultados reales — el dork exacto que usaste y qué encontraste
- [ ] Correr `theHarvester` contra `rappi.com` — documentar los emails encontrados e inferir el formato corporativo
- [ ] Encontrar un documento público de Rappi, descargarlo, y analizarlo con `exiftool` — documentar todos los campos que revelen algo

---

## Bonus — Más allá del recon básico

> [!TIP]
> No es obligatorio. Son tres ejercicios independientes — hacé el que más te llame la atención o los tres si tenés tiempo. Cada uno te lleva a una técnica que no vimos explícitamente en clase pero que aplica todo lo que sí vimos.

---

### B1. Lo que subfinder no encontró — Certificate Transparency

subfinder consulta fuentes de DNS pasivo. Pero hay otra fuente que pocos usan y que a veces encuentra subdominios que el resto no tiene: los **logs públicos de certificados SSL**.

Cada vez que una empresa emite un certificado TLS para un dominio o subdominio, ese evento queda registrado en logs públicos para siempre. No se puede borrar.

Buscá los certificados emitidos para Rappi directamente en:

```
https://crt.sh/?q=%.rappi.com
```

Comparalo con tu lista de subfinder:
- ¿Encontraste subdominios que subfinder **no** había encontrado?
- ¿Hay alguno que llame la atención por su nombre?
- ¿Hay certificados muy viejos? ¿Y certificados recientes para subdominios que ya no resuelven?

> [!NOTE]
> Un subdominio con certificado vigente que ya no resuelve puede ser interesante — alguien emitió un certificado para algo que después dejó de usar. En un pentest real eso vale investigarlo.

---

### B2. ¿Se puede falsificar un email de Rappi?

Ya sabés que Rappi usa cierto proveedor de correo (lo encontraste en el Nivel 1). Ahora la pregunta práctica es: **¿qué tan bien configurada está su seguridad de correo?**

Tres registros DNS definen si alguien puede suplantar el dominio de correo de una empresa:

```
SPF   → define desde qué servidores se puede enviar email legítimo de rappi.com
DKIM  → firma criptográfica que valida que el email no fue modificado
DMARC → política que indica qué hacer con los emails que fallan SPF o DKIM
```

Ya tenés el SPF del Nivel 1. Ahora buscá DMARC:

```bash
dig TXT _dmarc.rappi.com
```

Analizá el resultado con MXToolbox:
```
https://mxtoolbox.com/dmarc/rappi.com
https://mxtoolbox.com/spf/rappi.com
```

Respondé:
- ¿Tiene DMARC configurado?
- Si tiene DMARC, ¿cuál es la política? (`none` / `quarantine` / `reject`)
- Con `p=none` un atacante puede enviar emails que parecen venir de `@rappi.com` y llegar a la bandeja de entrada. ¿Es ese el caso?
- ¿Qué significa esto para un ataque de phishing dirigido a clientes o empleados de Rappi?

> [!NOTE]
> `p=none` en DMARC significa que la empresa configuró el registro pero solo está monitoreando — no bloqueando. Es la configuración más común y la más peligrosa desde el punto de vista del atacante.

---

### B3. Subdomain takeover — ¿hay algo abandonado?

Cuando una empresa usa un servicio en la nube (GitHub Pages, Heroku, S3, Netlify, etc.), crea un registro CNAME que apunta a ese servicio. Si después cancela el servicio pero **no borra el CNAME**, el subdominio queda apuntando a algo que ya no existe y que cualquiera puede reclamar.

Eso se llama **subdomain takeover** — es una vulnerabilidad real y es 100% detectable con recon pasivo.

De tu lista de subdominios activos del Nivel 2, buscá los que tienen registros CNAME:

```bash
# Para cada subdominio interesante de tu lista:
dig CNAME subdominio.rappi.com

# O procesar la lista completa:
cat subs_vivos.txt | dnsx -cname -resp -silent
```

Para cada CNAME que encuentres, fijate adónde apunta:
- ¿Termina en `.herokuapp.com`, `.github.io`, `.s3.amazonaws.com`, `.netlify.app`, `.azurewebsites.net`?
- Si abrís ese CNAME en el navegador, ¿responde con un error del tipo "No such app" o "404 - Repository not found"?

Si encontrás uno así: **eso es un hallazgo crítico**. Documentarlo con captura y explicar por qué importa.

---

## Entrega

Publicar en **GitHub Discussions** con el título: `[Challenge M1] Tu nombre — Rappi`

```
## Informe — rappi.com

### Resumen
(2-3 líneas: qué encontraste y cuál es el hallazgo más importante)

### Nivel 1 — Reconocimiento base
(resultado de cada tarea con output real o captura)

### Nivel 2 — Superficie de ataque
(subdominios activos, top 3 justificados, hallazgos de Shodan)

### Nivel 3 — OSINT
(dorks usados, emails encontrados, análisis de metadatos)

### Bonus
(si aplica)

### El hallazgo más impactante
¿Qué fue lo más interesante que encontraste? ¿Por qué importa?
```

> [!NOTE]
> Un hallazgo sin captura no existe. Para cada cosa relevante que encontrés, tomá la captura en el momento — no después.

---

## Revisión

El lunes al inicio de la clase hacemos 15 minutos de revisión:
- Cada uno comenta su hallazgo más interesante
- Vemos qué encontró cada alumno que los demás no vieron
- Discutimos qué técnica fue más efectiva para este objetivo

---

*Cualquier duda antes del domingo: canal del curso.*
