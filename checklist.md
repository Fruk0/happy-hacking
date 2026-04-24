# Checklist — Módulo 1
## Reconocimiento Pasivo

---

## Objetivo de la fase

Construir el mapa de superficie del objetivo usando solo fuentes públicas.
Al terminar, tenés que poder responder: *¿qué tiene expuesto este objetivo y por dónde entraría un atacante?*

---

## Qué estás buscando

No técnicas. Información que tiene valor real como atacante:

- **Nameservers y registrar** — indican madurez de seguridad del dominio
- **Proveedor de correo** — vector de phishing, configuración SPF/DKIM
- **Servicios SaaS internos** — revelados por registros TXT (Atlassian, Microsoft, Google)
- **Subdominios olvidados** — dev, staging, test, internal, beta
- **IPs y rangos de red** — para cruzar con Shodan
- **ASN de la organización** — superficie completa de infraestructura propia
- **Emails corporativos** — formato, empleados expuestos, vectores de ataque
- **Documentos públicos con metadatos** — username interno, rutas de sistema, software
- **Servicios con versiones expuestas** — especialmente en subdominios secundarios
- **Paneles de administración** indexados por Google
- **Archivos que no deberían ser públicos** — `.env`, `.log`, `.bak`, `.sql`

---

## Checklist de ejecución

Ejecutar en este orden. Cada paso alimenta al siguiente.

```bash
TARGET="dominio-objetivo.com"
mkdir recon_$TARGET && cd recon_$TARGET
```

- [ ] **Whois del dominio** — `whois $TARGET` → anotar registrar, nameservers, fechas, emails si aparecen
- [ ] **Whois de la IP principal** — `whois $(dig A $TARGET +short | head -1)` → ASN, rango, organización dueña del servidor
- [ ] **DNS completo** — loop por tipo (A, AAAA, MX, NS, TXT, SOA) → registrar todos los resultados
- [ ] **Leer los TXT** — identificar servicios SaaS por los `include:` del SPF y las verificaciones de dominio
- [ ] **Anotar los nameservers** — del `dig NS $TARGET +short` → los necesitás para el paso siguiente
- [ ] **Intentar zone transfer** — `dig axfr @NAMESERVER $TARGET` → si funciona es crítico; si falla, anotarlo y seguir
- [ ] **Enumerar subdominios con subfinder** — `subfinder -d $TARGET -silent -o subs.txt`
- [ ] **Ampliar con amass** — `amass enum -passive -d $TARGET -timeout 5 -o subs_amass.txt` → combinar con subfinder
- [ ] **Filtrar subdominios interesantes** — `cat subs.txt | grep -E "api|admin|dev|staging|vpn|test|internal|beta"`
- [ ] **Resolver a IPs** — `cat subs.txt | dnsx -a -resp -silent -o subs_vivos.txt` → quedarse con los activos
- [ ] **Shodan** — `org:"Empresa"` y `hostname:$TARGET` → puertos abiertos, versiones, tecnologías
- [ ] **Google Dorks** — al menos: `site:*.$TARGET -www`, `site:$TARGET ext:env OR ext:log OR ext:bak`, `intitle:"index of" site:$TARGET`
- [ ] **Emails y usuarios** — `theHarvester -d $TARGET -b google,bing,yahoo -l 100` → formato de email, empleados expuestos
- [ ] **Metadatos** — Google Dork: `site:$TARGET filetype:pdf` → descargar uno, `exiftool doc.pdf`
- [ ] **Documentar cada hallazgo** — herramienta + output + por qué importa + captura de pantalla

---

## Señales de que encontraste algo valioso

Cuando aparece cualquiera de estos, **parar y documentar antes de seguir**:

🔴 Zone transfer exitosa — tenés el mapa completo de la infraestructura interna  
🔴 Archivo `.env`, `.bak`, `.sql` o `.config` indexado públicamente en Google  
🔴 `FilePath` o `Author` en metadatos revela un usuario interno del sistema  
🟠 Subdominio con patrón `dev.`, `staging.`, `test.`, `internal.`, `beta.` — seguridad típicamente más laxa  
🟠 Panel de administración accesible sin autenticación  
🟠 Servicio con versión desactualizada visible en Shodan  
🟠 Directorio abierto listando archivos (`intitle:"index of"`)  
🟡 Email de empleado técnico (sysadmin, devops, IT) — vector de spear phishing  
🟡 Registro TXT con herramientas internas (Atlassian, Okta, Salesforce)  
🟡 CNAME apuntando a servicio cloud no reclamado — posible subdomain takeover  

---

## Errores que hacen perder tiempo

**Pasar directo a subfinder sin hacer DNS primero.**
El DNS te dice el proveedor de mail, los SaaS internos, y los nameservers para zone transfer.
Si lo saltás, arrancás ciego.

**No guardar el output de cada herramienta.**
Lo que no está en un archivo no existió. Usá siempre `-o` o `>` para redirigir.
Reconstruir de memoria lo que corriste hace dos horas es tiempo perdido.

**Atascarse cuando algo no devuelve resultados.**
`Transfer failed.` es un dato. Shodan sin resultados es un dato. Anotarlo y seguir.
El rabbit hole de "por qué no funciona esta herramienta" consume tiempo que vale más en otra superficie.

**Enumerar sin priorizar.**
subfinder puede devolver 300 subdominios. No investigás los 300 — filtrás los interesantes y arrancás por ahí.
Un `dev.` olvidado vale más que cien instancias de CDN.

**No anotar por qué algo es relevante en el momento.**
Un subdominio marcado como "interesante" sin contexto va a parecer ruido al día siguiente.
Una línea de explicación alcanza: *"staging.tesla.com — resuelve a IP pública, responde HTTP, posiblemente sin autenticación"*.

---

## El criterio que define si terminaste

El recon pasivo no termina cuando corriste todas las herramientas.
Termina cuando podés responder: *¿qué tiene expuesto este objetivo y por dónde entraría un atacante?*

Los datos no son hallazgos. Un subdominio es un dato.
Un subdominio de staging con un framework desactualizado accesible sin autenticación es un hallazgo.
La diferencia está en el análisis, no en la cantidad de comandos que corriste.

Documentá ahora. El reporte se escribe con evidencia, no con memoria.
