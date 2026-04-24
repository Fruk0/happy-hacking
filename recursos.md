# Recursos — Módulo 1
## Reconocimiento Pasivo

> Para profundizar más allá de las clases.
> 🆓 gratuito · 💰 pago · 🔑 requiere registro

---

## Índice

1. [Herramientas usadas en clase](#1-herramientas-usadas-en-clase)
2. [Consultas y bases de datos públicas](#2-consultas-y-bases-de-datos-públicas)
3. [Motores de búsqueda de infraestructura](#3-motores-de-búsqueda-de-infraestructura)
4. [Práctica y laboratorios](#4-práctica-y-laboratorios)
5. [Lectura recomendada](#5-lectura-recomendada)
6. [Videos y canales](#6-videos-y-canales)
7. [Certificaciones relevantes](#7-certificaciones-relevantes)

---

## 1. Herramientas usadas en clase

### DNS

| Herramienta | Qué hace | Instalación |
|-------------|----------|-------------|
| `dig` | Consultas DNS completas | `sudo apt install dnsutils` |
| `host` | Consultas DNS simples | `sudo apt install dnsutils` |
| `whois` | Datos de registro de dominios e IPs | `sudo apt install whois` |
| `dnsx` | Resolver subdominios en escala | `sudo apt install dnsx` |

### Subdomain Enumeration

| Herramienta | Qué hace | Instalación |
|-------------|----------|-------------|
| `subfinder` | Enumeración pasiva de subdominios | `sudo apt install subfinder` |
| `amass` | OSINT + enumeración de subdominios | `sudo apt install amass` |

### OSINT y metadatos

| Herramienta | Qué hace | Instalación |
|-------------|----------|-------------|
| `theHarvester` | Emails, usuarios, subdominios | `sudo apt install theharvester` |
| `exiftool` | Metadatos de archivos | `sudo apt install libimage-exiftool-perl` |

---

## 2. Consultas y bases de datos públicas

### Whois y registro de dominios

| Recurso | Descripción | Link |
|---------|-------------|------|
| NIC Argentina 🆓 | Registro oficial de dominios `.ar` | https://nic.ar/buscar |
| ICANN Lookup 🆓 | Whois centralizado para dominios globales | https://lookup.icann.org |
| who.is 🆓 | Whois web con historial | https://who.is |
| DomainTools 🔑💰 | Whois histórico + reverse whois | https://whois.domaintools.com |

### DNS

| Recurso | Descripción | Link |
|---------|-------------|------|
| MXToolbox 🆓 | DNS lookup, MX check, verificación de blacklists | https://mxtoolbox.com |
| dnsdumpster 🆓 | DNS recon + mapa visual de infraestructura | https://dnsdumpster.com |
| ViewDNS 🆓 | Reverse IP, historial DNS, Whois | https://viewdns.info |
| SecurityTrails 🔑 | DNS histórico + subdominios | https://securitytrails.com |

### Certificados SSL — fuente de subdominios

| Recurso | Descripción | Link |
|---------|-------------|------|
| crt.sh 🆓 | Certificados SSL/TLS públicos — excelente fuente de subdominios | https://crt.sh |
| Certificate Transparency 🆓 | Logs oficiales de certificados | https://certificate.transparency.dev |

### IP y ASN

| Recurso | Descripción | Link |
|---------|-------------|------|
| bgp.he.net 🆓 | ASN lookup, rangos IP, relaciones BGP | https://bgp.he.net |
| ipinfo.io 🆓 | IP → ASN, organización, geolocalización | https://ipinfo.io |
| LACNIC 🆓 | Registro de IPs América Latina y Caribe | https://www.lacnic.net |
| ARIN 🆓 | Registro de IPs América del Norte | https://search.arin.net |

---

## 3. Motores de búsqueda de infraestructura

| Recurso | Descripción | Link |
|---------|-------------|------|
| Shodan 🔑🆓/💰 | Dispositivos y servicios expuestos en internet | https://shodan.io |
| Censys 🔑🆓/💰 | Escaneo de internet, certificados, servicios | https://search.censys.io |
| Fofa 🔑🆓/💰 | Alternativa a Shodan, muy completa | https://fofa.info |
| GreyNoise 🔑 | Contexto sobre IPs que escanean internet activamente | https://viz.greynoise.io |
| ZoomEye 🔑 | Alternativa a Shodan | https://www.zoomeye.org |

> **Para empezar:** Shodan con cuenta gratuita alcanza para el módulo y el challenge.
> Censys tiene tier académico — si tenés email universitario, vale pedirlo.

---

## 4. Práctica y laboratorios

### Targets diseñados para practicar

| Recurso | Qué practicar | Link |
|---------|---------------|------|
| zonetransfer.me 🆓 | Transferencia de zona AXFR — el que usamos en clase | https://digi.ninja/projects/zonetransfer.php |
| HackTheBox 🔑🆓/💰 | Máquinas y challenges con componente de recon | https://hackthebox.com |
| TryHackMe 🔑🆓/💰 | Rooms guiados de OSINT y recon | https://tryhackme.com |

### Rooms de TryHackMe recomendados para este módulo

| Room | Qué practica |
|------|--------------|
| OhSINT | OSINT básico — buen punto de partida |
| Google Dorking | Google Hacking paso a paso |
| Passive Reconnaissance | Recon pasivo completo |
| Shodan.io | Uso de Shodan desde cero |

---

## 5. Lectura recomendada

### Metodologías

| Recurso | Descripción | Link |
|---------|-------------|------|
| PTES 🆓 | Penetration Testing Execution Standard — estándar de la industria | http://www.pentest-standard.org |
| OWASP Testing Guide 🆓 | Guía de testing de aplicaciones web | https://owasp.org/www-project-web-security-testing-guide |
| MITRE ATT&CK — Reconnaissance 🆓 | Tácticas de reconocimiento documentadas por MITRE | https://attack.mitre.org/tactics/TA0043 |

### Documentación de las herramientas del módulo

| Herramienta | Link |
|-------------|------|
| Subfinder | https://docs.projectdiscovery.io/tools/subfinder |
| Amass | https://github.com/owasp-amass/amass/blob/master/doc/user_guide.md |
| theHarvester | https://github.com/laramies/theHarvester/wiki |
| ExifTool | https://exiftool.org/exiftool_pod.html |

---

## 6. Videos y canales

| Canal | Por qué vale la pena |
|-------|----------------------|
| **IppSec** (YouTube) | Writeups de HackTheBox — siempre arranca con recon real bien explicado |
| **TCM Security** (YouTube) | Cursos de pentest prácticos, muy accesibles para principiantes |
| **NahamSec** (YouTube) | Bug Bounty con mucho foco en recon — casos reales |
| **John Hammond** (YouTube) | CTFs + análisis de malware + recon |
| **STÖK** (YouTube) | Bug bounty con énfasis en recon y OSINT |
| **HackerOne Hacker101** (YouTube) | Bug bounty desde cero, formato estructurado |

---

## 7. Certificaciones relevantes

Lo que podés lograr después de este curso:

| Certificación | Emisor | Nivel | Recon en el examen |
|---------------|--------|:-----:|--------------------|
| **eJPT** | INE / eLearnSecurity | Principiante | Sí — recon básico |
| **PNPT** | TCM Security | Intermedio | Sí — recon completo |
| **OSCP** | Offensive Security | Avanzado | Sí — fundamental |
| CEH | EC-Council | Teórico | Sí — mayormente teórico |

> **Ruta recomendada:** eJPT primero para validar lo aprendido.
> Después PNPT o directo a OSCP según el ritmo que te propongas.

---

*¿Encontraste un recurso útil que no está acá? Compartilo en el canal del curso.*
