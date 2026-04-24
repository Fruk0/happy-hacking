# Mindset del Pentester

> Las herramientas cambian. El criterio no.
> Antes de ejecutar cualquier comando, necesitás entender por qué lo ejecutás.

---

## La única diferencia que importa

Un hacker y un pentester usan las mismas herramientas, tienen los mismos conocimientos, y piensan exactamente igual.

La diferencia es **el permiso y el reporte**.

Con autorización documentada → profesional con carrera, ingresos, y protección legal.
Sin autorización → delito tipificado en la Ley 26.388 de Delitos Informáticos.

Eso es todo. No hay diferencia técnica, solo ética y legal. En este curso hacemos todo con autorización y documentamos cada paso.

---

## Pensá como atacante. Actuá como profesional.

El trabajo de un pentester es pensar exactamente como piensa alguien que quiere comprometer un sistema — y encontrar las vulnerabilidades antes de que ese alguien llegue.

**Cuestioná cada supuesto.**
"Seguro que no tienen eso expuesto" no es análisis — es esperanza.
Verificá todo. Lo que parece obvio a veces es lo más crítico.

**Pensá en impacto, no en técnica.**
Un subdominio de staging no es interesante porque encontraste un subdominio.
Es interesante porque puede tener datos reales, autenticación laxa, y un framework desactualizado.
La técnica es el medio. El impacto es lo que va al reporte y lo que le importa al cliente.

**Documentá mientras trabajás.**
El reporte se escribe con evidencia, no con memoria.
Un hallazgo sin captura no existió.
Guardá el output de cada herramienta desde el primer comando.

**Sabé cuándo parar.**
Encontrar una vulnerabilidad crítica no es una invitación a explotarla sin límite.
El scope define hasta dónde llegás. Salirse del scope invalida el trabajo y expone al consultor legalmente.

---

## La pregunta correcta en cada fase

```
Recon pasivo    →  ¿Qué tiene expuesto este objetivo sin que yo toque nada?
Recon activo    →  ¿Qué está corriendo y en qué versión?
Acceso inicial  →  ¿Por dónde entro con el menor ruido posible?
Post-ex         →  ¿Cuánto daño real podría hacer un atacante desde acá?
Reporte         →  ¿Puede alguien que no estuvo ahí entender qué encontré y por qué importa?
```

Si no podés responder la pregunta de la fase en la que estás, falta información o falta análisis.

---

## Errores que cometen los que recién empiezan

**Acumular herramientas en vez de entender resultados.**
Tener 50 herramientas instaladas y no saber interpretar el output de ninguna no sirve.
Dominar `dig`, `subfinder`, y `theHarvester` es más valioso que tener todo instalado.

**Confundir actividad con progreso.**
Correr 10 herramientas sin leer los resultados no es recon — es ruido.
Cinco minutos analizando el output de una herramienta valen más que una hora corriendo comandos.

**No guardar evidencia.**
Todo lo que encontrás va a un archivo. Siempre. Sin excepción.
El cliente no acepta "lo vi pero no lo guardé".

**Saltarse el recon pasivo.**
El recon pasivo es menos impresionante que explotar una vulnerabilidad.
También es la diferencia entre un pentest fallido y uno exitoso.
Los que se lo saltan se pierden el subdominio de staging con la base de datos expuesta.

---

## Lo que este curso te prepara para lograr

- Trabajar como pentester junior o bug bounty hunter
- Rendir el **eJPT** (INE/eLearnSecurity) — primer certificado recomendado
- Rendir el **PNPT** (TCM Security) — certificado de pentest práctico
- Prepararte para el **OSCP** (Offensive Security) con base sólida

Lo que no te vamos a dar: atajos. No existen.

---

*Volvé a leer esto cuando sientas que estás corriendo comandos sin entender por qué.*
