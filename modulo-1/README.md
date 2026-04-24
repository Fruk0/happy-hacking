<div align="center">

# Módulo 1 — Reconocimiento Pasivo

</div>

---

## Contexto del engagement

Nexus Financial S.A. acaba de contratarnos para un pentest externo.
No sabemos nada de su infraestructura. No tenemos acceso a nada. No tocamos nada.

**La primera pregunta es: ¿qué tiene expuesto este objetivo sin que nosotros hagamos nada activo?**

Esa pregunta se responde con reconocimiento pasivo — consultando fuentes públicas que ya existen, sin enviar un solo paquete a los servidores del objetivo. La empresa no nos ve.

---

## Dónde estamos en el pentest

```
[ RECON PASIVO ]  ──▶  Recon Activo  ──▶  Acceso  ──▶  Post-Ex  ──▶  Reporte
       ▲
   Módulo 1
   Estamos acá
```

---

## Qué vas a poder hacer al terminar

- Extraer información de infraestructura de cualquier empresa usando solo fuentes públicas
- Ejecutar consultas DNS completas e interpretar cada registro
- Intentar transferencias de zona y entender por qué son críticas cuando funcionan
- Enumerar subdominios pasivamente con `subfinder`, `amass` y `dnsx`
- Mapear infraestructura expuesta con Shodan y Google Hacking
- Encontrar emails, usuarios, y documentos sensibles con `theHarvester` y `exiftool`
- Encadenar todo en un pipeline con AI

---

## Material del módulo

| Archivo | Para qué sirve |
|---------|---------------|
| [comandos.md](./comandos.md) | Referencia completa — referencia rápida arriba, detalle abajo |
| [checklist.md](./checklist.md) | Qué ejecutar, en qué orden, y cómo saber si encontraste algo valioso |
| [recursos.md](./recursos.md) | Links, herramientas online, certificaciones |
| [challenge.md](./challenge.md) | Enunciado del challenge — se activa al final de la Clase 2 |

---

## Herramientas del módulo

| Herramienta | Clase | Qué hace |
|-------------|:-----:|----------|
| `whois` | 1 | Datos de registro de dominios e IPs |
| `dig` | 1 | Consultas DNS completas |
| `host` | 1 | Consultas DNS simples y rápidas |
| `subfinder` | 2 | Enumeración pasiva de subdominios |
| `amass` | 2 | OSINT + enumeración de subdominios |
| `dnsx` | 2 | Resolver subdominios a IPs en escala |
| Shodan | 2 | Servicios e infraestructura expuesta |
| Google Dorks | 2 | Archivos y servicios indexados públicamente |
| `theHarvester` | 2 | Emails y usuarios expuestos |
| `exiftool` | 2 | Metadatos de archivos públicos |

---

## Setup — antes de arrancar

Verificar que todo está disponible:

```bash
which dig whois host subfinder amass dnsx theHarvester exiftool
```

Instalar lo que falte:

```bash
# Actualizar repos
sudo apt update

# Herramientas base
sudo apt install -y dnsutils whois libimage-exiftool-perl

# Herramientas de enumeración (repositorios de Kali)
sudo apt install -y subfinder amass dnsx theharvester

# Si alguna no está en los repos de tu versión de Kali, instalar via Go:
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest

# Agregar Go al PATH si las herramientas instaladas no se encuentran:
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.zshrc && source ~/.zshrc
```

---

*Módulo 2 → Reconocimiento activo: `nmap`, fingerprinting, detección de versiones*
