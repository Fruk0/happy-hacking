# Comandos — Módulo 1
## Reconocimiento Pasivo Completo

> Todos los comandos de este módulo son **pasivos**.
> Consultamos fuentes públicas — no enviamos tráfico al objetivo.
> La empresa no sabe que existimos.

---

## Cómo usar este archivo

**Referencia rápida** → expandí las secciones de abajo, copiá el comando, ejecutá.
**Detalle completo** → bajá a la sección correspondiente para entender cada flag y leer el output.

---

## ⚡ Referencia rápida

<details>
<summary><strong>Whois</strong></summary>

```bash
whois tesla.com
whois mercadolibre.com
whois x.x.x.x                         # whois de una IP — ASN y rango
whois -h whois.nic.ar dominio.com.ar   # para dominios .ar específicamente
```
🌐 Dominios `.ar` → https://nic.ar/buscar

</details>

<details>
<summary><strong>DNS — dig</strong></summary>

```bash
# Consulta completa
dig tesla.com ANY

# Por tipo — los que más importan
dig A     tesla.com +short    # IP del servidor
dig MX    tesla.com +short    # servidores de correo
dig NS    tesla.com +short    # nameservers  ← anotar para zone transfer
dig TXT   tesla.com           # SPF, DKIM, servicios SaaS internos

# Consultar contra un DNS público (cuando el local falla)
dig @8.8.8.8 tesla.com A
dig @1.1.1.1 tesla.com MX

# Loop — todos los tipos de una vez
for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ==="
  dig $tipo tesla.com +short
done
```

</details>

<details>
<summary><strong>DNS — host</strong></summary>

```bash
host tesla.com              # A + AAAA + MX de un vistazo
host -t MX  tesla.com       # solo MX
host -t NS  tesla.com       # solo NS
host x.x.x.x               # reverso: IP → dominio
```

</details>

<details>
<summary><strong>Transferencia de zona</strong></summary>

```bash
# Contra un objetivo real (va a fallar si está bien configurado — eso es esperable)
dig axfr @NAMESERVER dominio.com

# Demo educativa — este dominio existe para practicarlo, siempre funciona
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

</details>

<details>
<summary><strong>Subdomain enumeration</strong></summary>

```bash
subfinder -d tesla.com -silent
subfinder -d tesla.com -silent -o subs.txt

amass enum -passive -d tesla.com -timeout 5
amass enum -passive -d tesla.com -timeout 5 -o subs_amass.txt

# Combinar y deduplicar
cat subs.txt subs_amass.txt | sort -u > subs_total.txt

# Filtrar los más interesantes
cat subs_total.txt | grep -E "api|admin|dev|staging|vpn|mail|test|beta|internal"
```

</details>

<details>
<summary><strong>Resolver subdominios a IPs — dnsx</strong></summary>

```bash
cat subs.txt | dnsx -a -resp -silent
cat subs.txt | dnsx -a -resp -silent -o subs_vivos.txt

# Pipeline directo sin archivo intermedio
subfinder -d tesla.com -silent | dnsx -a -resp -silent
```

</details>

<details>
<summary><strong>Shodan — búsquedas desde el navegador</strong></summary>

```
org:"Tesla Motors"
org:"Tesla Motors" port:22
org:"Tesla Motors" port:3389
org:"Tesla Motors" product:"Apache"
org:"MercadoLibre"
hostname:tesla.com
hostname:*.tesla.com
ssl:"tesla.com"
net:x.x.x.x/24
country:"AR" product:"Apache" port:80
```
🌐 https://shodan.io

</details>

<details>
<summary><strong>Google Hacking</strong></summary>

```
site:*.tesla.com -www
site:tesla.com filetype:pdf
site:tesla.com filetype:xlsx OR filetype:csv
site:tesla.com ext:env OR ext:log OR ext:bak
site:tesla.com ext:sql OR ext:config
site:tesla.com inurl:login
site:tesla.com inurl:admin
site:tesla.com inurl:dashboard
intitle:"index of" site:tesla.com
site:tesla.com "DB_PASSWORD" OR "API_KEY" OR "SECRET"
```

</details>

<details>
<summary><strong>theHarvester</strong></summary>

```bash
theHarvester -d tesla.com -b google,bing -l 100
theHarvester -d tesla.com -b google,bing,yahoo -l 100 -f output_tesla
theHarvester -d mercadolibre.com -b google,bing -l 50
```

</details>

<details>
<summary><strong>ExifTool — metadatos</strong></summary>

```bash
# Buscar el PDF con Google Dork: site:tesla.com filetype:pdf
wget "URL-DEL-PDF" -O target.pdf
exiftool target.pdf
exiftool target.pdf | grep -iE "author|creator|company|filepath|user|producer"
```
**Qué buscar:** `Author` · `Creator` · `Company` · `FilePath` · `ModifyDate`

</details>

<details>
<summary><strong>Flujo completo — en orden</strong></summary>

```bash
TARGET="tesla.com"
mkdir recon_$TARGET && cd recon_$TARGET

# 1. Whois
whois $TARGET > whois_dominio.txt
whois $(dig A $TARGET +short | head -1) > whois_ip.txt

# 2. DNS completo
for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ===" >> dns_completo.txt
  dig $tipo $TARGET +short >> dns_completo.txt
done

# 3. Zone transfer
NS=$(dig NS $TARGET +short | head -1)
dig axfr @$NS $TARGET > zone_transfer.txt

# 4. Subdominios
subfinder -d $TARGET -silent -o subs_subfinder.txt
amass enum -passive -d $TARGET -timeout 5 -o subs_amass.txt
cat subs_subfinder.txt subs_amass.txt | sort -u > subs_total.txt
echo "[+] Subdominios únicos: $(wc -l < subs_total.txt)"

# 5. Resolver a IPs
cat subs_total.txt | dnsx -a -resp -silent -o subs_vivos.txt
echo "[+] Subdominios activos: $(wc -l < subs_vivos.txt)"

# 6. Emails y usuarios
theHarvester -d $TARGET -b google,bing,yahoo -l 100 -f harvester_output

# 7. Metadatos
# Google Dork: site:$TARGET filetype:pdf
# wget "URL" -O doc.pdf && exiftool doc.pdf > metadatos.txt
```

</details>

<details>
<summary><strong>Tablas de referencia rápida</strong></summary>

**Tipos de registros DNS**

| Registro | Qué es | Por qué importa |
|----------|--------|-----------------|
| `A` | Dominio → IPv4 | Dónde vive el servidor |
| `AAAA` | Dominio → IPv6 | Infraestructura moderna |
| `MX` | Servidores de correo | Proveedor de mail |
| `NS` | Nameservers | Para intentar zone transfer |
| `TXT` | Texto libre | SPF, DKIM, servicios SaaS internos |
| `CNAME` | Alias de otro dominio | Servicios cloud, subdominios |
| `SOA` | Info del servidor DNS primario | Admin, versión de zona |

**Proveedores de correo comunes por registro MX**

| Lo que ves en el MX | Proveedor |
|---------------------|-----------|
| `aspmx.l.google.com` | Google Workspace |
| `*.mail.protection.outlook.com` | Microsoft 365 |
| `mxa.mailgun.org` | Mailgun |
| `*.pphosted.com` | Proofpoint |
| `*.mimecast.com` | Mimecast |

**Patrones de subdominios que siempre priorizamos**

| Patrón | Por qué importa |
|--------|-----------------|
| `api.` | Lógica de negocio, endpoints sin protección de UI |
| `dev.` / `test.` | Seguridad típicamente más laxa que producción |
| `staging.` | Copia de producción, a veces con datos reales |
| `admin.` | Panel de administración |
| `vpn.` | Acceso remoto |
| `internal.` | Algo interno que no debería ser público |
| `beta.` | Sin hardening completo |

**Errores comunes y soluciones**

| Síntoma | Causa probable | Fix |
|---------|---------------|-----|
| `dig` timeout | DNS local caído | `dig @8.8.8.8 ...` |
| `dig ANY` devuelve vacío | Servidor ignora ANY | Loop por tipo por separado |
| `whois` "No server known" | Extensión de dominio rara | `-h whois.nic.ar` |
| AXFR falla en zonetransfer.me | Nameserver incorrecto | Usar `@nsztm1.digi.ninja` exacto |
| `subfinder` sin resultados | Dominio muy nuevo | Agregar `-all -timeout 30` |
| `amass` no termina | Sin timeout | Siempre usar `-timeout 5` |
| Go tools no encontradas | PATH incompleto | `export PATH=$PATH:$(go env GOPATH)/bin` |

</details>

---

## 📖 Referencia detallada

---

## Clase 1

---

## 1. Whois

Cuando alguien registra un dominio, esa información queda en una base de datos pública.
Whois es el protocolo que existe desde los años 80 para consultarla.

```bash
whois tesla.com
# Buscar:
#   Registrar       → quién administra el dominio
#                     MarkMonitor = empresa grande con gestión seria de dominios
#                     GoDaddy/Namecheap = registro personal o empresa pequeña
#   Creation Date   → antigüedad del dominio (dominio viejo = más infraestructura legacy)
#   Name Server     → nameservers autoritativos — ANOTARLOS, se usan para zone transfer
#   Registrant Email → a veces redactado por GDPR; si aparece, es un email corporativo directo
```

```bash
whois mercadolibre.com
# Empresa LATAM — los datos suelen ser más visibles que en corporaciones globales
```

```bash
whois 104.215.148.63
# Whois de una IP directamente
# Devuelve: organización dueña del bloque, ASN, rango completo, región
# Útil para entender a quién pertenece el servidor del objetivo
```

```bash
whois -h whois.arin.net 104.215.148.63
# -h = especificar el servidor whois manualmente
# Registros regionales:
#   ARIN   → IPs de América del Norte   (whois.arin.net)
#   LACNIC → IPs de América Latina      (whois.lacnic.net)
#   RIPE   → IPs de Europa              (whois.ripe.net)
#   APNIC  → IPs de Asia-Pacífico       (whois.apnic.net)
```

> **Dato de contexto:** Tesla registró tesla.com en 1992 — varios años antes de existir como fabricante de autos.
> El dominio original pertenecía a otra empresa. Eso lo revela el `Creation Date` de whois.

---

## 2. DNS con dig

`dig` (Domain Information Groper) es la herramienta principal para consultas DNS.
Viene instalada en Kali como parte del paquete `dnsutils`.

### Tipos de registros DNS

```
A      → Dominio → IPv4. Dónde vive el servidor.
AAAA   → Dominio → IPv6.
MX     → Mail Exchanger. Servidores de correo.
NS     → Name Server. Nameservers autoritativos del dominio.
TXT    → Texto libre: SPF, DKIM, verificaciones de SaaS.
CNAME  → Alias que apunta a otro nombre de dominio.
SOA    → Start of Authority: info del DNS primario, admin, versión de zona.
PTR    → Reverso: IP → dominio.
```

### Consultas básicas

```bash
dig tesla.com ANY
# ANY = dame todos los registros que tengas
# Algunos servidores modernos rechazan o ignoran ANY — si pasa, usar el loop más abajo
```

```bash
dig A tesla.com +short
# +short = solo el resultado, sin cabeceras ni metadata de la consulta
# Devuelve una o varias IPs
# Varias IPs = balanceo de carga o CDN
```

```bash
dig MX tesla.com +short
# Servidores de correo con su prioridad
# Número menor = mayor prioridad
# Ejemplo: "10 mxa.mailgun.org." → Tesla usa Mailgun
#
# Qué revelan los registros MX:
#   mxa/mxb.mailgun.org          → Mailgun (email transaccional externo)
#   *.mail.protection.outlook.com → Microsoft 365
#   aspmx.l.google.com           → Google Workspace
#   *.pphosted.com               → Proofpoint (gateway de seguridad de correo)
```

```bash
dig NS tesla.com +short
# Nameservers autoritativos
# ANOTARLOS: se usan para intentar transferencia de zona en el siguiente paso
```

```bash
dig TXT tesla.com
# Sin +short para ver el contenido completo
# Qué buscar en los TXT:
#
#   "v=spf1 include:mailgun.org ~all"
#   → Usa Mailgun para enviar correo. Confirma el proveedor de mail.
#
#   "v=spf1 include:_spf.google.com ~all"
#   → Usa Google Workspace.
#
#   "MS=msXXXXXXXX"
#   → Tiene cuenta de Microsoft 365.
#
#   "atlassian-domain-verification=XXXX"
#   → Usa Confluence o Jira (Atlassian Cloud).
#
#   "google-site-verification=XXXX"
#   → Tiene Google Search Console o Google Ads activos.
#
# Cada verificación es una herramienta interna expuesta sin querer.
```

### Consultar contra un servidor DNS específico

```bash
dig @8.8.8.8 tesla.com A
# @8.8.8.8 = usar Google DNS como resolver en vez del DNS local
# Útil cuando el DNS de la red local filtra o da resultados incorrectos

dig @1.1.1.1 tesla.com MX
# Cloudflare DNS como alternativa a Google
```

### Loop para consultar todos los tipos a la vez

```bash
for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ==="
  dig $tipo tesla.com +short
done
# Alternativa cuando dig ANY no devuelve resultados completos
# Cada tipo en su propia consulta — más confiable con servidores modernos
```

### Cómo leer el output completo de dig

```
;; QUESTION SECTION:     → lo que preguntamos
;; ANSWER SECTION:       → la respuesta — acá está la data que importa
;; AUTHORITY SECTION:    → qué servidor es autoritativo para este dominio
;; ADDITIONAL SECTION:   → información extra (IPs de los nameservers)

;; Query time: 45 msec   → latencia al servidor DNS
;; SERVER: 8.8.8.8#53    → qué servidor respondió a la consulta
```

---

## 3. DNS con host

`host` es más directo que `dig`. Útil para resultados limpios y rápidos.

```bash
host tesla.com
# Muestra registros A, AAAA y MX en una sola línea cada uno
# Sin cabeceras — output limpio, ideal para demo
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
# Equivalente a: dig -x 104.215.148.63
```

**¿Cuándo usar cada uno?**

```
dig   → análisis detallado, TTL, flags, output que se puede procesar con scripts
host  → resultado rápido, demo limpia, verificación rápida de un registro
```

---

## 4. Transferencia de zona

Una transferencia de zona (AXFR) sincroniza todos los registros DNS de un servidor primario a uno secundario. Es una función legítima del protocolo DNS.

**El problema:** si el servidor está mal configurado, acepta el pedido de cualquiera y entrega el mapa completo de toda la infraestructura interna.

```bash
# Contra un objetivo real — va a fallar si está bien configurado
dig axfr @ns1.markmonitor.com tesla.com
# axfr  = tipo de consulta para full zone transfer
# @ns1.markmonitor.com = nameserver al que le preguntamos (obtenido del dig NS anterior)
# tesla.com = dominio que queremos transferir
#
# RESULTADO ESPERADO: "Transfer failed." → el servidor rechaza la petición
# Ese fallo es una buena noticia para Tesla — significa que está bien configurado
```

```bash
# Demo educativa — siempre funciona
dig axfr @nsztm1.digi.ninja zonetransfer.me
# zonetransfer.me es un dominio creado específicamente para practicar esto
# Está configurado para PERMITIR la transferencia — existe para demostraciones
# SIEMPRE usar este nameserver exacto: nsztm1.digi.ninja
```

**Output esperado de zonetransfer.me (parcial):**

```
zonetransfer.me.          7200  IN  A      5.196.105.14
email.zonetransfer.me.    7200  IN  A      74.125.206.26
office.zonetransfer.me.   7200  IN  A      4.23.39.254
vpn.zonetransfer.me.     14400  IN  A      174.36.59.154
staging.zonetransfer.me.  7200  IN  CNAME  www.zonetransfer.me.
```

**Por qué es crítico cuando funciona:**

```
Sin zone transfer:   subfinder + amass + horas de enumeración  →  lista parcial
Con zone transfer:   un solo comando                           →  mapa completo
```

> **En un pentest real:** siempre intentar zone transfer antes de pasar a enumeración de subdominios.
> Si funciona, nos ahorramos horas. Si falla, lo anotamos como dato y seguimos.

---

## 5. NIC Argentina — dominios .ar

Los dominios `.ar` no están en registrars internacionales.
Los administra NIC Argentina. El `whois` clásico desde terminal puede ser incompleto o fallar.

**Fuente canónica para dominios .ar:** https://nic.ar/buscar

```bash
# Intentar desde terminal (resultado variable según el dominio)
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

> Para engagements contra empresas argentinas — bancos, telcos, utilities — siempre pasar por nic.ar.
> Es un paso que se omite en muchos cursos y donde se pierde información que estaba disponible.

---

## Clase 2

---

## 6. Subdomain enumeration — subfinder

subfinder consulta fuentes pasivas en paralelo sin tocar el objetivo: certificados SSL públicos, bases de datos de DNS pasivo, APIs de threat intelligence.

```bash
subfinder -d tesla.com -silent
# -d      = dominio objetivo
# -silent = sin banner ni logs de progreso — solo los subdominios encontrados
```

```bash
subfinder -d tesla.com -silent -o subs_tesla.txt
# -o = guardar output en archivo
# Siempre guardar — se usa con dnsx y para filtrar después
```

```bash
subfinder -d tesla.com -silent | wc -l
# Contar cuántos subdominios encontró
```

```bash
subfinder -d tesla.com -all -silent -timeout 30
# -all     = usar todas las fuentes disponibles, incluyendo las que requieren API key
# -timeout = segundos máximos por fuente (30 es un buen balance)
# Más lento, más completo — usar cuando el resultado estándar parece escaso
```

```bash
# Filtrar los subdominios más interesantes del output
cat subs_tesla.txt | grep -E "api|admin|dev|staging|vpn|mail|test|beta|internal"
# grep -E = extended regex — permite usar | como OR
# Estos patrones son los que más frecuentemente tienen seguridad más laxa
```

---

## 7. Subdomain enumeration — amass

amass consulta más fuentes que subfinder pero es considerablemente más lento. Incluye análisis de ASN, Whois, certificados, y múltiples fuentes de DNS pasivo.

```bash
amass enum -passive -d tesla.com -timeout 5
# -passive = solo fuentes pasivas, sin fuerza bruta de nombres
# -timeout = minutos máximos de ejecución — sin esto puede correr por horas
#            5 minutos es el mínimo recomendado para cobertura útil
```

```bash
amass enum -passive -d mercadolibre.com -timeout 5 -o subs_amass.txt
# -o = guardar output
```

```bash
# Combinar resultados de subfinder y amass y eliminar duplicados
cat subs_tesla.txt subs_amass.txt | sort -u > subs_total.txt
echo "[+] Subdominios únicos: $(wc -l < subs_total.txt)"
```

**subfinder vs amass:**

```
subfinder   → segundos, buen punto de partida, ideal para triage inicial
amass       → minutos, más fuentes, mejor cobertura final
Uso real    → correr ambos, combinar resultados con sort -u
```

---

## 8. Resolver subdominios a IPs — dnsx

subfinder y amass devuelven nombres — algunos históricos que ya no existen.
dnsx resuelve cada subdominio y filtra los que tienen resolución activa.

```bash
cat subs_total.txt | dnsx -a -resp -silent
# -a      = consultar registros A (IPv4)
# -resp   = mostrar la respuesta completa (subdominio → IP)
# -silent = sin banner
```

```bash
cat subs_total.txt | dnsx -a -resp -silent -o subs_vivos.txt
# Guardar solo los que resolvieron — son la superficie de ataque real
```

```bash
# Ver cuántos están activos del total
echo "[+] Activos: $(wc -l < subs_vivos.txt) de $(wc -l < subs_total.txt)"
```

```bash
# Pipeline completo en un paso
subfinder -d tesla.com -silent | dnsx -a -resp -silent
```

> Los subdominios que no resuelven son históricos o internos.
> Solo los que resolvieron son superficie de ataque accesible desde internet.

---

## 9. Shodan

Shodan indexa dispositivos y servicios conectados a internet. No indexa páginas web — indexa lo que está expuesto en los puertos: versión del servicio, certificado SSL, banners, tecnología.

**Acceso:** https://shodan.io — cuenta gratuita alcanza para el curso.

### Búsquedas por organización

```
org:"Tesla Motors"
# Todo lo que Shodan tiene indexado de Tesla
# Muestra: cantidad de hosts, puertos más frecuentes, países, tecnologías

org:"MercadoLibre"
org:"Telecom Argentina"
```

### Filtros de puerto y servicio

```
org:"Tesla Motors" port:22      → SSH expuesto
org:"Tesla Motors" port:3389    → RDP expuesto (Windows)
org:"Tesla Motors" port:8080    → web alternativa, paneles de admin
org:"Tesla Motors" port:443     → HTTPS
org:"Tesla Motors" product:"Apache"
org:"Tesla Motors" product:"nginx"
org:"Tesla Motors" http.title:"admin"
```

### Búsquedas por dominio y certificado

```
hostname:tesla.com
hostname:*.tesla.com
ssl:"tesla.com"      → busca por certificado SSL — encuentra subdominios no enumerados
```

### Infraestructura por IP o rango

```
net:x.x.x.x/24              → todos los hosts en ese rango de IPs
```

### Infraestructura argentina

```
country:"AR" product:"Apache" port:80
org:"Telecom Argentina" port:22
org:"Banco Galicia"
```

**Cómo leer un resultado:**

```
IP: 198.51.100.10
Puertos abiertos: 22, 80, 443

80/tcp  → Apache httpd 2.4.29
443/tcp → Apache httpd 2.4.29 (SSL)
22/tcp  → OpenSSH 7.4

Qué revela:
  → Apache 2.4.29 — versión con CVEs conocidos documentados
  → OpenSSH 7.4   — versión vieja, potencialmente vulnerable
  → SSH abierto   — vector de acceso remoto
Todo esto sin haber enviado un paquete al servidor.
```

---

## 10. Google Hacking

Google indexa todo lo que encuentra públicamente. Incluyendo cosas que las empresas no querían que Google encontrara. Los dorks usan los operadores avanzados de Google para buscar exactamente eso.

### Mapear subdominios y estructura

```
site:*.tesla.com -www
# site: = solo resultados de este dominio y sus subdominios
# *. = wildcard — cualquier subdominio
# -www = excluir www (ya lo conocemos)
```

### Archivos expuestos

```
site:tesla.com filetype:pdf
# PDFs públicos — investor relations, manuales, documentos internos subidos sin querer

site:tesla.com filetype:xlsx OR filetype:csv
# Planillas — a veces con datos operativos

site:tesla.com ext:env OR ext:log OR ext:bak
# .env  = archivos de configuración con variables de entorno
#         a veces contienen credenciales, API keys, passwords de base de datos
# .log  = logs de aplicación — revelan stack, errores, usuarios
# .bak  = backups — pueden ser copias del código fuente
```

### Paneles de acceso

```
site:tesla.com inurl:login
site:tesla.com inurl:admin
site:tesla.com inurl:dashboard
site:tesla.com inurl:portal
```

### Directorios abiertos

```
intitle:"index of" site:tesla.com
# "index of" = título por defecto de Apache cuando directory listing está habilitado
# Si aparece: el servidor lista sus archivos sin autenticación
```

### Credenciales y datos sensibles

```
site:tesla.com "DB_PASSWORD" OR "API_KEY" OR "SECRET"
site:tesla.com "password" filetype:env
```

### Encontrar PDFs para analizar con exiftool

```
site:tesla.com filetype:pdf inurl:report
site:tesla.com filetype:pdf inurl:investor
site:tesla.com filetype:pdf inurl:manual
```

> Si encontrás algo sensible en una empresa real durante el challenge — documentarlo, no tocarlo,
> reportarlo en la entrega explicando el hallazgo. Eso es comportamiento profesional.

---

## 11. theHarvester

theHarvester busca emails, usuarios, y subdominios a través de motores de búsqueda y fuentes públicas. Mapea el factor humano del objetivo.

```bash
theHarvester -d tesla.com -b google,bing -l 100
# -d = dominio objetivo
# -b = fuentes de búsqueda separadas por coma
# -l = límite de resultados por fuente
```

```bash
theHarvester -d tesla.com -b google,bing,yahoo -l 100 -f output_tesla
# -f = guardar output (genera output_tesla.xml y output_tesla.json)
# yahoo a veces devuelve emails que google y bing no tienen
```

```bash
theHarvester -d mercadolibre.com -b google,bing -l 50
```

**Fuentes disponibles:**

```
google       → Google Search
bing         → Bing Search
yahoo        → Yahoo Search
dnsdumpster  → DNS recon pasivo — útil como complemento
linkedin     → LinkedIn (perfiles de empleados — limitado sin API)
hunter       → Hunter.io (emails verificados — requiere API key)
```

**Qué hacer con los emails encontrados:**

```
1. Identificar el formato: nombre.apellido@ / n.apellido@ / nombre@
2. Inferir emails de empleados que no aparecen públicamente con ese formato
3. Documentar: "la empresa tiene X emails corporativos expuestos públicamente"
4. En fase de acceso (M3): vector de spear phishing — fuera del scope de M1
```

---

## 12. ExifTool — metadatos

Los archivos tienen metadatos generados automáticamente por el software que los creó: nombre del autor, software usado, fecha, y a veces la ruta interna del sistema donde fue guardado. Esa información sale a internet dentro del archivo y nadie lo nota.

```bash
# Paso 1 — encontrar un PDF público con Google Dork:
# site:tesla.com filetype:pdf

# Paso 2 — descargarlo
wget "https://URL-DEL-PDF" -O target.pdf
# wget = descargador de línea de comandos
# -O   = nombre del archivo de salida

# Paso 3 — analizar todos los metadatos
exiftool target.pdf

# Paso 4 — filtrar los campos que más revelan
exiftool target.pdf | grep -iE "author|creator|company|filepath|user|producer"
```

```bash
# Analizar varios archivos a la vez
exiftool *.pdf
exiftool *.docx

# Output en JSON (para scripting o documentación)
exiftool -json target.pdf

# Extraer campos específicos
exiftool -Author -Creator -Company target.pdf
```

**Los campos que más revelan:**

```
Author          → nombre del empleado que creó el documento
                  posible username de Active Directory o email corporativo

Creator         → software usado internamente
                  versión de Microsoft Office, sistema operativo inferible

Company         → nombre real de la empresa o subsidiaria
                  a veces difiere del nombre comercial conocido

FilePath        → ruta interna del sistema donde fue guardado
                  ejemplo: C:\Users\jsmith\Documents\IR\Q4_2024\final_v3.pdf
                  revela: usuario de Windows ("jsmith"), estructura de carpetas,
                  versioning del documento (final_v3 = hubo v1, v2, v3)

ModifyDate      → cuándo fue modificado por última vez
                  puede revelar zona horaria del sistema operativo
```

> El campo `FilePath` es el más valioso: revela el nombre de usuario de Windows del empleado
> que creó el documento — posiblemente su usuario de dominio en Active Directory.

---

## 13. Flujo completo encadenado

Orden de ejecución para un recon pasivo completo. Cada paso alimenta al siguiente.

```bash
# ══════════════════════════════════════════════════
# SETUP
# ══════════════════════════════════════════════════
TARGET="tesla.com"
mkdir recon_$TARGET && cd recon_$TARGET

# ══════════════════════════════════════════════════
# PASO 1 — WHOIS
# Qué buscamos: registrar, nameservers, fechas
# ══════════════════════════════════════════════════
whois $TARGET > whois_dominio.txt

IP=$(dig A $TARGET +short | head -1)
whois $IP > whois_ip.txt

# ══════════════════════════════════════════════════
# PASO 2 — DNS COMPLETO
# Qué buscamos: proveedor de mail, SaaS internos, nameservers
# ══════════════════════════════════════════════════
for tipo in A AAAA MX NS TXT SOA; do
  echo "=== $tipo ===" >> dns_completo.txt
  dig $tipo $TARGET +short >> dns_completo.txt
done

# ══════════════════════════════════════════════════
# PASO 3 — ZONE TRANSFER
# Si funciona: mapa completo. Si falla: dato para el reporte.
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
# PASO 5 — RESOLVER A IPs (filtrar los activos)
# ══════════════════════════════════════════════════
cat subs_total.txt | dnsx -a -resp -silent -o subs_vivos.txt
echo "[+] Activos: $(wc -l < subs_vivos.txt) de $(wc -l < subs_total.txt)"

# ══════════════════════════════════════════════════
# PASO 6 — PRIORIZAR SUBDOMINIOS INTERESANTES
# ══════════════════════════════════════════════════
cat subs_vivos.txt | grep -E "api|admin|dev|staging|vpn|mail|test|beta|internal"

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
echo "Archivos generados   : $(ls *.txt | wc -l)"
echo "════════════════════════════════════"
ls -lh
```

---

## 14. Errores comunes y soluciones

### `dig` no responde / timeout

```bash
# Causa: resolver DNS local caído o filtrando consultas
dig @8.8.8.8 tesla.com A
dig @1.1.1.1 tesla.com MX
```

### `dig ANY` devuelve vacío o SERVFAIL

```bash
# Causa: el servidor ignora consultas ANY (comportamiento normal en DNS modernos)
# Solución: consultar cada tipo por separado
for tipo in A MX NS TXT; do echo "=== $tipo ==="; dig $tipo tesla.com +short; done
```

### `whois` devuelve "No whois server is known"

```bash
# Causa: extensión de dominio poco común o servidor whois no detectado
whois -h whois.nic.ar dominio.com.ar
```

### Zone transfer falla en zonetransfer.me

```bash
# Causa casi siempre: estás usando el nameserver incorrecto
# Solución: usar siempre este nameserver exacto
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

### `subfinder` no encuentra subdominios

```bash
# Causa: dominio muy nuevo o poca presencia pública en fuentes pasivas
subfinder -d TARGET.com -all -silent -timeout 30
```

### `amass` no termina nunca

```bash
# Causa: sin timeout, amass puede correr durante horas
amass enum -passive -d TARGET.com -timeout 5
# -timeout = minutos máximos de ejecución
```

### `theHarvester` devuelve pocos o ningún resultado

```bash
# Causa: Google limita scraping — resultados variables por horario y IP
# Solución: agregar más fuentes
theHarvester -d TARGET.com -b bing,yahoo,dnsdumpster -l 100
```

### `dnsx` no resuelve nada

```bash
# Verificar que el archivo de entrada tiene un subdominio por línea
head -5 subs_tesla.txt
# Probar con un subdominio conocido
echo "api.tesla.com" | dnsx -a -resp -silent
```

### Las herramientas instaladas con Go no se encuentran

```bash
# Causa: $(go env GOPATH)/bin no está en el PATH
export PATH=$PATH:$(go env GOPATH)/bin
# Para que persista en todas las sesiones:
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.zshrc && source ~/.zshrc
```

---

*Módulo 2 → Reconocimiento activo: `nmap`, detección de versiones, fingerprinting de servicios*
