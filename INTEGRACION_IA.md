Registro de Trabajo con Inteligencia Artificial (Requerimiento Obligatorio)

### DBA - Plataforma Conecta Rural

Este documento certifica el uso estratégico, la guía y la validación de herramientas de Inteligencia Artificial en el diseño de la solución técnica, cumpliendo con los lineamientos de transparencia exigidos por la pauta.

---

## Bitácora de Co-Diseño y Dirección Técnica

A lo largo del proceso de desarrollo de este MVP (con un límite sugerido de 3 horas), la interacción con la IA no se utilizó para copiar código de forma ciega, sino como un **copiloto de arquitectura** para resolver dos desafíos principales:

### 1. Resolución de Contingencias de Infraestructura Local

* **Problema:** La estación de trabajo local operaba bajo un sistema macOS El Capitan obsoleto, bloqueando la instalación nativa de arquitecturas modernas de Docker Desktop y provocando fallos de negociación SSL/TLS (`invalid certificate chain`) al conectar con las APIs actuales de GitHub.

* **Dirección Humana:** Se instruyó a la IA para trazar una estrategia alternativa. La IA propuso el uso de nubes virtuales (*GitHub Codespaces*), pero ante la imposibilidad de renderizar la web de GitHub en navegadores antiguos de la Mac, el operador guio la solución hacia la terminal nativa, forzando la desactivación temporal de la verificación SSL en Git (`http.sslVerify false`) y automatizando la creación del repositorio público mediante comandos directos por SSH/HTTPS con Tokens de Acceso (PAT) generados por canal secundario.

### 2. Diseño del Negocio y Mitigación del Error (PITR)

* **Problema:** El requerimiento exigía simular un desastre (un `UPDATE` sin `WHERE`) pero forzaba a "rescatar" la data legítima ingresada *posterior* al incidente, un escenario no cubierto por los restores físicos planos.

* **Dirección Humana:** Se desafió a la IA a diseñar un mecanismo que impidiera la destrucción de datos huérfanos. Bajo supervisión del operador, se co-diseñó la **Estrategia de Aislamiento Lógico Pre-Restauración**, programando bloques en Bash que aíslan de forma selectiva las inserciones críticas mediante filtros de marcas de tiempo a archivos planos CSV antes de que pgBackRest reescriba los bloques físicos mediante Write-Ahead Logging (WAL).

---

## Verificación y Transparencia del Diálogo

La conversación completa con el modelo de lenguaje de Google abarcó los siguientes hitos verificables reflejados en el repositorio:
1. **Fase de Diagnóstico:** Análisis del contexto institucional de la Secretaría de Gobierno Digital (Transición del servicio a la Subsecretaría de Hacienda, perfiles técnicos e ingeniería).
2. **Fase de Hacks Operacionales:** Superación de restricciones de red, llaves criptográficas y cadenas de certificados SSL inválidas en macOS antiguo.
3. **Fase de Robustecimiento:** Estructuración del script `demo.sh` (con los RUTs secuenciales de la pauta) y creación de `demo2.sh` (con inyección de volumen y RUTs de validación real chilena).
4. **Fase de Negocio:** Formulación analítica de los indicadores de RPO y RTO incluidos en el README principal.

El registro interactivo demuestra una metodología de desarrollo donde el humano ejerce el rol de arquitecto y validador, asegurando la consistencia legal y técnica requerida por el servicio público.
