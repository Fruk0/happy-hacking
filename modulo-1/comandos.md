# Comandos — Módulo 1
## Reconocimiento Pasivo Completo

> Todos los comandos de este módulo son **pasivos**.
> Consultamos fuentes públicas — no enviamos tráfico al objetivo.
> La empresa no sabe que existimos.

---

## Cómo usar este archivo

**Referencia rápida** → expandí las secciones colapsables de abajo, copiá el comando, ejecutá.
**Detalle completo** → cada sección tiene teoría, comandos comentados, qué buscar en el output, y un ejercicio para practicar antes de seguir.

---

## ⚡ Referencia rápida

<details>
<summary><strong>Whois</strong></summary>

```bash
whois tesla.com
whois mercadolibre.com
whois x.x.x.x                         # whois de una IP — ASN y rango
whois -h whois.nic.ar dominio.com.ar   # para dominios .ar
```
🌐 Dominios `.ar` → https://nic.ar/buscar

</details>

<details>
<summary><strong>DNS — dig</strong></summary>

```bash
dig A     tesla.com +short
dig MX    tesla.com +short
dig NS    tesla.com +short
dig TXT   tesla.com
dig @8.8.8.8 tesla.com A

for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ==="
  dig $tipo tesla.com +short
done
```

</details>

<details>
<summary><strong>DNS — host</strong></summary>

```bash
host tesla.com
host -t MX  tesla.com
host -t NS  tesla.com
host x.x.x.x
```

</details>

<details>
<summary><strong>Transferencia de zona</strong></summary>

```bash
dig axfr @NAMESERVER dominio.com
dig axfr @nsztm1.digi.ninja zonetransfer.me   # demo educativa
```

</details>

<details>
<summary><strong>Subdomain enumeration</strong></summary>

```bash
subfinder -d tesla.com -silent -o subs.txt
amass enum -passive -d tesla.com -timeout 5 -o subs_amass.txt
cat subs.txt subs_amass.txt | sort -u > subs_total.txt
cat subs_total.txt | grep -E "api|admin|dev|staging|vpn|mail|test|beta|internal"
```

</details>

<details>
<summary><strong>Resolver subdominios a IPs — dnsx</strong></summary>

```bash
cat subs.txt | dnsx -a -resp -silent -o subs_vivos.txt
subfinder -d tesla.com -silent | dnsx -a -resp -silent
```

</details>

<details>
<summary><strong>Shodan</strong></summary>

```
org:"Tesla Motors"
org:"Tesla Motors" port:22
org:"Tesla Motors" product:"Apache"
hostname:tesla.com
ssl:"tesla.com"
country:"AR" product:"Apache" port:80
```
🌐 https://shodan.io

</details>

<details>
<summary><strong>Google Hacking</strong></summary>

```
site:*.tesla.com -www
site:tesla.com filetype:pdf
site:tesla.com ext:env OR ext:log OR ext:bak
site:tesla.com inurl:admin
intitle:"index of" site:tesla.com
site:tesla.com "DB_PASSWORD" OR "API_KEY"
```

</details>

<details>
<summary><strong>theHarvester</strong></summary>

```bash
theHarvester -d tesla.com -b google,bing -l 100
theHarvester -d tesla.com -b google,bing,yahoo -l 100 -f output_tesla
```

</details>

<details>
<summary><strong>ExifTool</strong></summary>

```bash
wget "URL-DEL-PDF" -O target.pdf
exiftool target.pdf
exiftool target.pdf | grep -iE "author|creator|company|filepath|user"
```

</details>

<details>
<summary><strong>Flujo completo</strong></summary>

```bash
TARGET="tesla.com"
mkdir recon_$TARGET && cd recon_$TARGET
whois $TARGET > whois_dominio.txt
for tipo in A AAAA MX NS TXT SOA; do echo "=== $tipo ===" >> dns.txt; dig $tipo $TARGET +short >> dns.txt; done
NS=$(dig NS $TARGET +short | head -1) && dig axfr @$NS $TARGET > zone_transfer.txt
subfinder -d $TARGET -silent -o subs_subfinder.txt
amass enum -passive -d $TARGET -timeout 5 -o subs_amass.txt
cat subs_subfinder.txt subs_amass.txt | sort -u > subs_total.txt
cat subs_total.txt | dnsx -a -resp -silent -o subs_vivos.txt
theHarvester -d $TARGET -b google,bing,yahoo -l 100 -f harvester_output
```

</details>

<details>
<summary><strong>Tablas de referencia</strong></summary>

**Tipos de registros DNS**

| Registro | Qué es | Por qué importa |
|----------|--------|-----------------|
| `A` | Dominio → IPv4 | Dónde vive el servidor |
| `MX` | Servidores de correo | Proveedor de mail |
| `NS` | Nameservers | Para intentar zone transfer |
| `TXT` | Texto libre | SPF, DKIM, servicios SaaS internos |
| `CNAME` | Alias de otro dominio | Servicios cloud, subdominios |
| `SOA` | Info del DNS primario | Admin, versión de zona |

**Proveedores de correo por registro MX**

| Lo que ves | Proveedor |
|------------|-----------|
| `aspmx.l.google.com` | Google Workspace |
| `*.mail.protection.outlook.com` | Microsoft 365 |
| `mxa.mailgun.org` | Mailgun |
| `*.pphosted.com` | Proofpoint |
| `*.mimecast.com` | Mimecast |

**Subdominios que siempre priorizamos**

| Patrón | Por qué importa |
|--------|-----------------|
| `api.` | Endpoints sin protección de UI |
| `dev.` / `test.` | Seguridad más laxa que producción |
| `staging.` | Copia de producción, a veces con datos reales |
| `admin.` | Panel de administración |
| `vpn.` | Acceso remoto |
| `internal.` | Algo interno que no debería ser público |

</details>

---

## 📖 Referencia detallada

---

# Clase 1

---

## 1. Whois

Cuando alguien registra un dominio, esa información queda en una base de datos pública. Whois es el protocolo que permite consultarla — existe desde los años 80 y es la primera herramienta que ejecutamos contra cualquier objetivo.

Para nosotros como atacantes, Whois revela datos de la organización antes de tocar cualquier otra herramienta: quién administra el dominio, desde cuándo existe, y —lo más importante— cuáles son sus nameservers.

```bash
whois tesla.com
# Consulta los datos de registro del dominio
```

```bash
whois 104.215.148.63
# Whois de una IP — a quién pertenece ese bloque de IPs
# Devuelve: organización dueña, ASN, rango completo, región geográfica
```

```bash
whois -h whois.nic.ar personal.com.ar
# -h = especificar el servidor whois manualmente
# Para dominios .ar, apuntar directo al servidor de NIC Argentina
```

> [!NOTE]
> **Qué buscar en el output:**
>
> - **Registrar** → quién administra el dominio. *MarkMonitor* indica gestión corporativa seria. *GoDaddy* o *Namecheap* indican registro más informal.
> - **Creation Date** → antigüedad del dominio. Un dominio viejo implica más infraestructura legacy y potencialmente más superficie histórica.
> - **Name Server** → nameservers autoritativos. **Anotarlos** — son necesarios para intentar la transferencia de zona.
> - **Registrant Email** → frecuentemente redactado por GDPR. Si aparece, es un email corporativo directo.

> [!TIP]
> Después de obtener la IP principal del dominio con `dig A tesla.com +short`, hacé `whois` sobre esa IP también. Te da el ASN y el rango de IPs completo — útil para búsquedas en Shodan más adelante.

> [!WARNING]
> El whois clásico desde terminal puede ser incompleto o fallar con dominios `.ar`. Para esos casos la fuente correcta es https://nic.ar/buscar en el navegador.

---

> 🔧 **Intentalo**
>
> Ejecutá `whois tesla.com` y anotá en un archivo:
> 1. El nombre del registrar
> 2. Los nameservers — los vas a necesitar en la sección 4
> 3. La fecha de creación
>
> Después hacé lo mismo con `whois mercadolibre.com`. ¿Qué diferencias encontrás entre los dos?

---

> ❓ ¿Qué dato de Whois tenés que guardar obligatoriamente antes de pasar a la transferencia de zona?

---

## 2. DNS con dig

El Sistema de Nombres de Dominio (DNS) traduce nombres legibles (`tesla.com`) a IPs que las computadoras entienden. Para nosotros, DNS es un mapa — cada tipo de registro es una pista distinta sobre la infraestructura del objetivo.

`dig` (Domain Information Groper) es la herramienta principal para hacer estas consultas. Viene instalada en Kali como parte del paquete `dnsutils`.

### Tipos de registros

```
A      → Dominio → IPv4. Dónde vive el servidor.
AAAA   → Dominio → IPv6.
MX     → Mail Exchanger. Servidores de correo.
NS     → Name Server. Nameservers autoritativos.
TXT    → Texto libre: SPF, DKIM, verificaciones de SaaS.
CNAME  → Alias que apunta a otro nombre de dominio.
SOA    → Start of Authority: info del DNS primario.
PTR    → Reverso: IP → dominio.
```

### Consultas básicas

```bash
dig A tesla.com +short
# +short = solo el resultado, sin cabeceras
# Una IP = servidor dedicado
# Varias IPs = balanceo de carga o CDN
```

```bash
dig MX tesla.com +short
# Servidores de correo con su prioridad
# Número menor = mayor prioridad
# Ejemplo: "10 mxa.mailgun.org." → Tesla usa Mailgun
```

```bash
dig NS tesla.com +short
# Nameservers autoritativos
# ANOTARLOS — se usan para zone transfer en la sección 4
```

```bash
dig TXT tesla.com
# Sin +short para ver el contenido completo
# Los TXT pueden ser largos — +short los trunca
```

```bash
dig @8.8.8.8 tesla.com A
# @ = especificar el resolver DNS
# Usar cuando el DNS local falla o da resultados incorrectos
# 8.8.8.8 = Google DNS    1.1.1.1 = Cloudflare DNS
```

```bash
# Loop — consultar todos los tipos de una vez
for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ==="
  dig $tipo tesla.com +short
done
# Más confiable que dig ANY, que muchos servidores modernos rechazan
```

> [!NOTE]
> **Cómo leer los registros TXT:**
>
> Los registros TXT revelan qué servicios usa la empresa internamente, sin haberlos publicado deliberadamente.
>
> | Lo que ves en el TXT | Qué significa |
> |----------------------|---------------|
> | `v=spf1 include:mailgun.org` | Usa Mailgun para enviar correo |
> | `v=spf1 include:_spf.google.com` | Usa Google Workspace |
> | `MS=msXXXXXX` | Tiene cuenta de Microsoft 365 |
> | `atlassian-domain-verification=XXX` | Usa Jira o Confluence |
> | `google-site-verification=XXX` | Tiene Google Search Console activo |
>
> Cada verificación es una herramienta interna expuesta sin intención.

> [!TIP]
> **Cómo leer el output completo de dig** (sin `+short`):
> ```
> ;; QUESTION SECTION:   → lo que preguntamos
> ;; ANSWER SECTION:     → la respuesta — acá está la data
> ;; AUTHORITY SECTION:  → qué servidor es autoritativo
> ;; ADDITIONAL SECTION: → IPs de los nameservers
> ;; Query time: 45 msec → latencia al servidor DNS
> ```

> [!IMPORTANT]
> `dig tesla.com ANY` — aunque parece conveniente — muchos servidores modernos lo rechazan o devuelven respuestas incompletas. Siempre preferir el loop por tipo para resultados confiables.

---

> 🔧 **Intentalo**
>
> Ejecutá contra `tesla.com` y contra `mercadolibre.com`:
> ```bash
> dig MX tesla.com +short
> dig TXT tesla.com
> ```
> Antes de seguir respondé:
> 1. ¿Qué proveedor de correo usa Tesla?
> 2. ¿Qué servicios SaaS podés identificar en los TXT?

---

> ❓ Encontraste `"10 aspmx.l.google.com."` en el registro MX de una empresa. ¿Qué proveedor de correo usa?

---

## 3. DNS con host

`host` es más directo que `dig`. Devuelve el resultado sin cabeceras ni metadata — útil para resultados limpios y rápidos.

```bash
host tesla.com
# Devuelve registros A, AAAA y MX en una sola línea cada uno
```

```bash
host -t MX mercadolibre.com
# -t = tipo de registro específico
# Equivalente a: dig MX mercadolibre.com +short
```

```bash
host -t NS personal.com.ar
# Nameservers de un dominio .ar
```

```bash
host 104.215.148.63
# Resolución inversa: IP → nombre de dominio
```

> [!NOTE]
> **¿Cuándo usar `dig` y cuándo usar `host`?**
>
> | Situación | Herramienta |
> |-----------|-------------|
> | Análisis completo, TTL, flags | `dig` |
> | Resultado rápido para un registro | `host` |
> | Scripting o procesamiento posterior | `dig` |
> | Demo limpia o captura de pantalla | `host` |

---

> 🔧 **Intentalo**
>
> Usá `host -t MX` contra tres empresas distintas y anotá el proveedor de correo de cada una. ¿Cuántas usan Google Workspace? ¿Cuántas Microsoft 365?

---

## 4. Transferencia de zona

Una transferencia de zona (AXFR) es un mecanismo legítimo del protocolo DNS: permite que un servidor secundario sincronice todos los registros desde el servidor primario. En un entorno bien configurado, solo los servidores autorizados pueden pedirla.

**El problema:** si el servidor está mal configurado, acepta la solicitud de cualquier IP y entrega el mapa completo de la infraestructura DNS — todos los subdominios, todas las IPs, todos los servicios internos.

```bash
# Contra un objetivo real — va a fallar si está bien configurado
dig axfr @ns1.markmonitor.com tesla.com
# axfr                 = tipo de consulta para full zone transfer
# @ns1.markmonitor.com = nameserver al que le pedimos la transferencia
#                        (obtenido del dig NS anterior)
# tesla.com            = dominio del que queremos todos los registros
#
# Resultado esperado: "Transfer failed." → bien configurado, es lo correcto
```

```bash
# Demo educativa — este dominio existe para practicar esto
dig axfr @nsztm1.digi.ninja zonetransfer.me
# zonetransfer.me está configurado deliberadamente para PERMITIR la transferencia
# Fue creado para demostraciones — siempre usar este nameserver exacto
```

**Output esperado de `zonetransfer.me`:**

```
zonetransfer.me.          7200  IN  A      5.196.105.14
email.zonetransfer.me.    7200  IN  A      74.125.206.26
office.zonetransfer.me.   7200  IN  A      4.23.39.254
vpn.zonetransfer.me.     14400  IN  A      174.36.59.154
staging.zonetransfer.me.  7200  IN  CNAME  www.zonetransfer.me.
```

> [!IMPORTANT]
> **¿Por qué es crítico cuando funciona?**
>
> ```
> Sin zone transfer:   subfinder + amass + horas  →  lista parcial e incompleta
> Con zone transfer:   un solo comando             →  mapa completo garantizado
> ```
>
> En un pentest real, siempre intentar zone transfer **antes** de pasar a enumeración de subdominios. Si funciona, nos ahorramos horas y el mapa es completo.

> [!NOTE]
> Una zone transfer exitosa en producción es una vulnerabilidad **crítica**. Expone toda la infraestructura interna, incluyendo subdominios que nunca fueron publicados intencionalmente.

> [!WARNING]
> Si el AXFR falla contra `zonetransfer.me`, casi siempre es porque estás usando el nameserver incorrecto. El único que funciona es `@nsztm1.digi.ninja` — exactamente así, sin variaciones.

---

> 🔧 **Intentalo**
>
> 1. Ejecutá el AXFR contra tesla.com con los nameservers que anotaste en la sección 1. Verificá que falla y anotá el mensaje de error exacto.
> 2. Ejecutá el AXFR contra `zonetransfer.me`. Listá los tres subdominios más interesantes que encontraste y explicá por qué.

---

> ❓ La zone transfer falló contra el objetivo. ¿Eso es un fracaso o un dato útil? ¿Qué anotás en el reporte?

---

## 5. NIC Argentina — dominios .ar

Los dominios `.ar` no están gestionados por registrars internacionales. Los administra NIC Argentina, el organismo oficial del espacio de nombres del país. El `whois` clásico desde terminal puede dar información incompleta o fallar directamente.

```bash
# Intentar desde terminal — resultado variable
whois personal.com.ar

# Apuntar directamente al servidor de NIC Argentina
whois -h whois.nic.ar personal.com.ar
```

```bash
# Las consultas DNS funcionan exactamente igual que con cualquier dominio
dig NS  personal.com.ar +short
dig MX  personal.com.ar +short
dig A   personal.com.ar +short
dig TXT personal.com.ar

host -t MX personal.com.ar
```

> [!TIP]
> Para dominios `.ar`, la fuente canónica es https://nic.ar/buscar. Ingresá el dominio y obtenés: titular registrado, nameservers, fechas y estado — más completo que el whois desde terminal.

> [!NOTE]
> En engagements contra empresas argentinas — bancos, telcos, utilities, organismos públicos — siempre pasar por NIC Argentina. Es un paso que se omite frecuentemente y donde se pierde información disponible públicamente.

---

> 🔧 **Intentalo**
>
> Buscá `personal.com.ar` en https://nic.ar/buscar. Compará la información que obtenés ahí con la que devuelve `whois personal.com.ar` desde la terminal. ¿Qué diferencias encontrás?

---

---

# Clase 2

---

## 6. Subdomain enumeration — subfinder

Hasta ahora trabajamos con el dominio raíz. Pero una empresa tiene decenas o cientos de subdominios, cada uno potencialmente corriendo un servicio diferente con un equipo diferente a cargo.

`subfinder` enumera subdominios consultando fuentes pasivas en paralelo: registros de certificados SSL, bases de datos de DNS pasivo, y APIs de threat intelligence. No toca los servidores del objetivo.

```bash
subfinder -d tesla.com -silent
# -d      = dominio objetivo
# -silent = sin banner, solo los subdominios encontrados
```

```bash
subfinder -d tesla.com -silent -o subs_tesla.txt
# -o = guardar en archivo
# Siempre guardar — se usa con dnsx y para filtrar después
```

```bash
subfinder -d tesla.com -silent | wc -l
# Contar cuántos subdominios encontró
```

```bash
subfinder -d tesla.com -all -silent -timeout 30
# -all     = todas las fuentes disponibles
# -timeout = segundos máximos por fuente
# Más lento, más completo
```

```bash
cat subs_tesla.txt | grep -E "api|admin|dev|staging|vpn|mail|test|beta|internal"
# grep -E = extended regex — | actúa como OR
# Filtra los subdominios que típicamente tienen seguridad más laxa
```

> [!NOTE]
> **Patrones de subdominios que siempre priorizamos:**
>
> | Patrón | Por qué importa |
> |--------|-----------------|
> | `api.` | Lógica de negocio, endpoints sin protección de UI |
> | `dev.` / `test.` | Seguridad típicamente más laxa que producción |
> | `staging.` | Copia de producción, a veces con datos reales |
> | `admin.` | Panel de administración |
> | `vpn.` | Acceso remoto |
> | `internal.` | Algo interno que no debería ser público |
> | `beta.` | Sin hardening completo |

> [!TIP]
> subfinder no hace fuerza bruta — no adivina nombres. Consulta bases de datos que ya los tienen registrados: certificados SSL (crt.sh), DNS pasivo (SecurityTrails), APIs de inteligencia (VirusTotal, Shodan). Por eso es 100% pasivo.

> [!WARNING]
> Un resultado de 0 subdominios casi siempre significa que el dominio es muy nuevo o tiene poca presencia pública. Probá con `-all -timeout 30` antes de asumir que no hay nada.

---

> 🔧 **Intentalo**
>
> 1. Ejecutá `subfinder -d tesla.com -silent -o subs_tesla.txt`
> 2. Contá cuántos encontró con `wc -l subs_tesla.txt`
> 3. Filtrá los interesantes con el grep del ejemplo
> 4. Anotá los 3 que más te llamen la atención y explicá por qué

---

> ❓ subfinder encontró 250 subdominios. ¿Cuáles investigás primero y por qué?

---

## 7. Subdomain enumeration — amass

`amass` es más completo que subfinder pero considerablemente más lento. Además de las fuentes de DNS pasivo, analiza relaciones de ASN, datos de Whois, y múltiples APIs de threat intelligence.

```bash
amass enum -passive -d tesla.com -timeout 5
# -passive = solo fuentes pasivas, sin fuerza bruta
# -timeout = minutos máximos — SIN ESTO PUEDE CORRER POR HORAS
```

```bash
amass enum -passive -d mercadolibre.com -timeout 5 -o subs_amass.txt
# -o = guardar output
```

```bash
# Combinar resultados de subfinder y amass, eliminar duplicados
cat subs_tesla.txt subs_amass.txt | sort -u > subs_total.txt
echo "[+] Subdominios únicos: $(wc -l < subs_total.txt)"
```

> [!NOTE]
> **¿Cuándo usar cada herramienta?**
>
> | Herramienta | Cuándo usarla |
> |-------------|---------------|
> | `subfinder` | Triage inicial, resultados en segundos |
> | `amass` | Cobertura máxima, cuando tenés tiempo |
> | Ambas combinadas | Pentest real — siempre |

> [!WARNING]
> **Nunca corras `amass` sin `-timeout`.** Sin ese flag puede correr durante horas. El mínimo recomendado es `-timeout 5` (5 minutos).

---

> 🔧 **Intentalo**
>
> Corré `amass enum -passive -d mercadolibre.com -timeout 5`. Mientras corre, abrí otra terminal y avanzá a la siguiente sección. Cuando termine, combiná el output con el de subfinder.

---

## 8. Resolver subdominios a IPs — dnsx

subfinder y amass devuelven nombres — muchos pueden ser históricos y ya no existir. `dnsx` resuelve cada subdominio y filtra los que tienen resolución activa hoy. Los que resuelven son la superficie de ataque real.

```bash
cat subs_total.txt | dnsx -a -resp -silent
# -a      = consultar registros A (IPv4)
# -resp   = mostrar subdominio → IP
# -silent = sin banner
```

```bash
cat subs_total.txt | dnsx -a -resp -silent -o subs_vivos.txt
echo "[+] Activos: $(wc -l < subs_vivos.txt) de $(wc -l < subs_total.txt)"
```

```bash
# Pipeline directo
subfinder -d tesla.com -silent | dnsx -a -resp -silent
```

> [!IMPORTANT]
> La diferencia entre subdominios totales y subdominios activos es normal. Un ratio de 20-40% activos sobre el total encontrado es esperable. Lo que importa es la lista de activos — esos son los servicios reales expuestos hoy.

> [!TIP]
> Un subdominio que no resuelve desde internet puede resolver internamente en la red de la empresa. Anotá los que no resolvieron también — durante el pentest interno pueden ser relevantes.

---

> 🔧 **Intentalo**
>
> Resolvé tu lista combinada de `tesla.com` con `dnsx`:
> 1. ¿Cuántos estaban activos del total encontrado?
> 2. ¿Aparece `api.tesla.com`? ¿A qué IP resuelve?

---

> ❓ Tenés 300 subdominios en la lista. dnsx resuelve 80. ¿Qué hacés con los 220 que no resolvieron?

---

## 9. Shodan

Shodan es un motor de búsqueda que indexa dispositivos y servicios conectados a internet. No indexa páginas web — indexa lo que está expuesto en los puertos: versión del servicio, sistema operativo, certificado SSL, banners, tecnología.

**Acceso:** https://shodan.io — cuenta gratuita alcanza para el curso y el challenge.

### Por organización

```
org:"Tesla Motors"
org:"MercadoLibre"
org:"Telecom Argentina"
```

### Por puerto y servicio

```
org:"Tesla Motors" port:22        → SSH expuesto
org:"Tesla Motors" port:3389      → RDP expuesto (acceso remoto Windows)
org:"Tesla Motors" port:8080      → web alternativa, paneles de administración
org:"Tesla Motors" product:"Apache"
org:"Tesla Motors" http.title:"admin"
```

### Por dominio y certificado SSL

```
hostname:tesla.com
hostname:*.tesla.com
ssl:"tesla.com"    → encuentra subdominios no enumerados via certificado SSL
```

### Infraestructura argentina

```
country:"AR" product:"Apache" port:80
org:"Telecom Argentina" port:22
```

> [!NOTE]
> **Cómo leer un resultado de Shodan:**
>
> ```
> IP: 198.51.100.10    Puertos: 22, 80, 443
>
> 80/tcp  → Apache httpd 2.4.29
> 22/tcp  → OpenSSH 7.4
> ```
>
> Qué revela:
> - Apache 2.4.29 → versión con CVEs documentados públicamente
> - OpenSSH 7.4 → versión vieja, potencialmente vulnerable
> - SSH abierto → vector de acceso remoto expuesto
>
> Todo sin haber enviado un paquete al servidor.

> [!TIP]
> La búsqueda `ssl:"dominio.com"` a veces encuentra subdominios que ni subfinder ni amass detectaron — porque el certificado los referencia aunque no aparezcan en DNS pasivo. Siempre complementar la enumeración con esta búsqueda.

---

> 🔧 **Intentalo**
>
> Buscá `org:"MercadoLibre"` en Shodan:
> 1. ¿Cuántos hosts encontró?
> 2. ¿Qué puertos aparecen con mayor frecuencia?
> 3. Aplicá el filtro `port:22`. ¿Cuántos servidores tienen SSH expuesto?

---

> ❓ Encontraste en Shodan un servidor del objetivo con Apache 2.4.29 en el puerto 80. ¿Es un hallazgo o solo un dato? ¿Qué diferencia hay?

---

## 10. Google Hacking

Google indexa todo lo que encuentra públicamente. Incluyendo cosas que las empresas no querían que indexara. Los dorks usan los operadores avanzados de búsqueda para encontrar exactamente eso — lo que no debería estar ahí.

### Mapear subdominios

```
site:*.tesla.com -www
```

### Archivos expuestos

```
site:tesla.com filetype:pdf
site:tesla.com filetype:xlsx OR filetype:csv
site:tesla.com ext:env OR ext:log OR ext:bak
site:tesla.com ext:sql OR ext:config
```

### Paneles de acceso

```
site:tesla.com inurl:login
site:tesla.com inurl:admin
site:tesla.com inurl:dashboard
```

### Directorios abiertos

```
intitle:"index of" site:tesla.com
```

### Datos sensibles

```
site:tesla.com "DB_PASSWORD" OR "API_KEY" OR "SECRET"
site:tesla.com "password" filetype:env
```

### PDFs para analizar con exiftool

```
site:tesla.com filetype:pdf inurl:report
site:tesla.com filetype:pdf inurl:investor
```

> [!IMPORTANT]
> El dork `ext:env OR ext:log OR ext:bak` es el de mayor impacto cuando devuelve resultados. Un archivo `.env` con credenciales es un hallazgo **crítico** en cualquier metodología, y en bug bounty significa pago inmediato.

> [!NOTE]
> Cuando usamos dorks no estamos tocando al objetivo — estamos leyendo el índice que Google ya construyó visitando esas páginas. El objetivo no recibe tráfico de nuestra parte.

> [!CAUTION]
> Si durante el challenge encontrás un archivo real con credenciales de una empresa real: documentarlo, no abrirlo, reportarlo en la entrega explicando el hallazgo. Ese es el comportamiento profesional esperado.

---

> 🔧 **Intentalo**
>
> Ejecutá estos dorks contra el dominio de tu empresa asignada en el challenge:
> 1. `site:*.DOMINIO -www` → ¿cuántos subdominios indexó Google?
> 2. `site:DOMINIO filetype:pdf` → descargá uno para usarlo en la sección 12
> 3. `intitle:"index of" site:DOMINIO` → ¿aparece algún directorio abierto?

---

> ❓ El dork `site:tesla.com ext:env` devuelve resultados. ¿Cuál es el primer paso antes de hacer clic en cualquiera de esos links?

---

## 11. theHarvester

Hasta acá mapeamos infraestructura técnica. Ahora mapeamos el factor humano: emails corporativos, usuarios expuestos, personas que son vectores de ataque potenciales.

`theHarvester` busca emails, usuarios y subdominios a través de motores de búsqueda y fuentes públicas.

```bash
theHarvester -d tesla.com -b google,bing -l 100
# -d = dominio objetivo
# -b = fuentes separadas por coma
# -l = límite de resultados por fuente
```

```bash
theHarvester -d tesla.com -b google,bing,yahoo -l 100 -f output_tesla
# -f = guardar output — genera output_tesla.xml y output_tesla.json
# yahoo a veces devuelve emails que google y bing no tienen
```

**Fuentes disponibles:**

```
google       → Google Search
bing         → Bing Search
yahoo        → Yahoo Search
dnsdumpster  → DNS recon pasivo
hunter       → Hunter.io (emails verificados — requiere API key)
```

> [!NOTE]
> **Qué hacer con los emails encontrados:**
>
> 1. Identificar el formato → `nombre.apellido@` / `n.apellido@` / `nombre@`
> 2. Inferir emails de empleados que no aparecen públicamente usando ese formato
> 3. Documentar la exposición en el reporte
> 4. En fase de acceso (Módulo 3): vector de spear phishing

> [!WARNING]
> Google limita el scraping activamente. Si theHarvester devuelve muy pocos resultados con `-b google`, probá con `-b bing,yahoo,dnsdumpster` como alternativa.

---

> 🔧 **Intentalo**
>
> Ejecutá `theHarvester -d tesla.com -b google,bing -l 100`. Antes de seguir:
> 1. ¿Cuántos emails encontró?
> 2. ¿Podés identificar el formato de email que usa Tesla?
> 3. Con ese formato, ¿podrías inferir el email de algún empleado conocido?

---

> ❓ theHarvester encontró `press@tesla.com` e `investor.relations@tesla.com`. ¿Qué te dice sobre la estructura de comunicaciones de la empresa?

---

## 12. ExifTool — metadatos

Cuando alguien crea un archivo en su computadora de trabajo, ese archivo guarda silenciosamente información del sistema: quién lo creó, con qué software, cuándo, y a veces la ruta interna del disco. Esa información viaja dentro del archivo cuando se publica en internet, y nadie lo nota.

```bash
# Paso 1: encontrar un PDF con Google Dork → site:tesla.com filetype:pdf

# Paso 2: descargarlo
wget "https://URL-DEL-PDF" -O target.pdf

# Paso 3: analizar todos los metadatos
exiftool target.pdf

# Paso 4: filtrar los campos más reveladores
exiftool target.pdf | grep -iE "author|creator|company|filepath|user|producer"
```

```bash
exiftool *.pdf            # analizar varios archivos a la vez
exiftool -json target.pdf # output en JSON para documentación
```

> [!NOTE]
> **Los campos que más revelan:**
>
> | Campo | Qué revela |
> |-------|------------|
> | `Author` | Nombre del empleado → posible username de Active Directory |
> | `Creator` | Software interno → versión de Office, SO inferible |
> | `Company` | Nombre real de la empresa o subsidiaria |
> | `FilePath` | Ruta del sistema: `C:\Users\jsmith\Documents\...` → usuario Windows, estructura de carpetas |
> | `ModifyDate` | Última modificación → puede revelar zona horaria del sistema |

> [!IMPORTANT]
> El campo `FilePath` es el más valioso. Revela el nombre de usuario Windows del empleado que creó el documento — posiblemente su usuario de dominio en Active Directory. Un PDF de investor relations puede exponer la estructura de carpetas interna sin que nadie lo haya notado.

> [!TIP]
> No te limitás a PDFs. Los archivos `.docx`, `.xlsx` y `.pptx` también tienen metadatos ricos. Buscalos con:
> ```
> site:tesla.com filetype:docx OR filetype:xlsx OR filetype:pptx
> ```

---

> 🔧 **Intentalo**
>
> Tomá el PDF que descargaste en la sección de Google Hacking y ejecutá `exiftool` sobre él.
> 1. ¿Encontraste un `Author`?
> 2. ¿Hay un `FilePath` o `Creator` que revele software o usuario interno?
> 3. ¿Qué conclusiones podés sacar de esos metadatos?

---

> ❓ exiftool devuelve `FilePath: C:\Users\mgarcia\Documents\Legal\contracts_2024\final.pdf`. ¿Qué información útil extraés de esa sola línea?

---

## 13. Flujo completo encadenado

Orden de ejecución para un recon pasivo completo. Cada paso alimenta al siguiente — no saltearse ninguno.

```bash
# ══════════════════════════════════════════════════
# SETUP
# ══════════════════════════════════════════════════
TARGET="tesla.com"
mkdir recon_$TARGET && cd recon_$TARGET

# ══════════════════════════════════════════════════
# PASO 1 — WHOIS
# ══════════════════════════════════════════════════
whois $TARGET > whois_dominio.txt
whois $(dig A $TARGET +short | head -1) > whois_ip.txt

# ══════════════════════════════════════════════════
# PASO 2 — DNS COMPLETO
# ══════════════════════════════════════════════════
for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ===" >> dns_completo.txt
  dig $tipo $TARGET +short >> dns_completo.txt
done

# ══════════════════════════════════════════════════
# PASO 3 — ZONE TRANSFER
# ══════════════════════════════════════════════════
NS=$(dig NS $TARGET +short | head -1)
dig axfr @$NS $TARGET > zone_transfer.txt
cat zone_transfer.txt

# ══════════════════════════════════════════════════
# PASO 4 — SUBDOMINIOS
# ══════════════════════════════════════════════════
subfinder -d $TARGET -silent -o subs_subfinder.txt
amass enum -passive -d $TARGET -timeout 5 -o subs_amass.txt
cat subs_subfinder.txt subs_amass.txt | sort -u > subs_total.txt
echo "[+] Subdominios únicos: $(wc -l < subs_total.txt)"

# ══════════════════════════════════════════════════
# PASO 5 — RESOLVER A IPs
# ══════════════════════════════════════════════════
cat subs_total.txt | dnsx -a -resp -silent -o subs_vivos.txt
echo "[+] Activos: $(wc -l < subs_vivos.txt) de $(wc -l < subs_total.txt)"

# ══════════════════════════════════════════════════
# PASO 6 — PRIORIZAR
# ══════════════════════════════════════════════════
echo "[+] Subdominios de alto interés:"
cat subs_vivos.txt | grep -E "api|admin|dev|staging|vpn|test|beta|internal"

# ══════════════════════════════════════════════════
# PASO 7 — EMAILS Y USUARIOS
# ══════════════════════════════════════════════════
theHarvester -d $TARGET -b google,bing,yahoo -l 100 -f harvester_output

# ══════════════════════════════════════════════════
# PASO 8 — METADATOS
# Google Dork: site:$TARGET filetype:pdf → copiar URL
# ══════════════════════════════════════════════════
# wget "URL" -O doc.pdf
# exiftool doc.pdf | grep -iE "author|creator|company|filepath|user"

# ══════════════════════════════════════════════════
# RESUMEN
# ══════════════════════════════════════════════════
echo ""
echo "════════════════════════════════════"
echo "RECON COMPLETADO — $TARGET"
echo "════════════════════════════════════"
echo "Subdominios totales  : $(wc -l < subs_total.txt)"
echo "Subdominios activos  : $(wc -l < subs_vivos.txt)"
echo "════════════════════════════════════"
ls -lh
```

> [!IMPORTANT]
> Este flujo es la base de todo recon pasivo profesional. En engagements reales lo vas a adaptar — más fuentes, más herramientas, más tiempo. Pero el orden siempre es el mismo: Whois → DNS → Zone Transfer → Subdominios → Resolución → Emails → Metadatos.

---

## 14. Errores comunes y soluciones

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| `dig` timeout | DNS local caído | `dig @8.8.8.8 tesla.com A` |
| `dig ANY` vacío | Servidor rechaza ANY | Loop por tipo por separado |
| `whois` "No server known" | Extensión de dominio rara | `whois -h whois.nic.ar dominio.com.ar` |
| AXFR falla en zonetransfer.me | Nameserver incorrecto | Usar `@nsztm1.digi.ninja` exacto |
| `subfinder` devuelve 0 | Dominio muy nuevo | `subfinder -d TARGET -all -timeout 30` |
| `amass` no termina | Falta `-timeout` | `amass enum -passive -d TARGET -timeout 5` |
| `theHarvester` devuelve poco | Google limitando scraping | Cambiar a `-b bing,yahoo,dnsdumpster` |
| `dnsx` no resuelve nada | Archivo vacío o formato incorrecto | `head -5 subs.txt` para verificar |
| Go tools no encontradas | PATH incompleto | `export PATH=$PATH:$(go env GOPATH)/bin` |

> [!TIP]
> Si algo no está en esta tabla: corré el comando con `-v` o `--debug` si está disponible y leé el mensaje de error completo antes de buscar solución. El 80% de los errores tienen la causa explicada en el output.

---

*Módulo 2 → Reconocimiento activo: `nmap`, detección de versiones, fingerprinting de servicios*
