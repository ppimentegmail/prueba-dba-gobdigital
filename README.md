Prueba  Técnica DBA: Point-In-Time Recovery (PITR) con pgBackRest
### Ecosistema Tecnológico - Plataforma "Conecta Rural"

Este repositorio contiene la arquitectura conceptual y la simulación técnica automatizada para mitigar pérdidas de datos lógicas (como sentencias `UPDATE` sin cláusula `WHERE`) en el sistema de subsidios Conecta Rural, implementando archivado continuo de WAL (Write-Ahead Logging) y PITR mediante **pgBackRest**.

## Resumen de la Solución y Supuestos de Arquitectura

Para efectos de agilidad institucional y desarrollo rápido de este MVP bajo un límite estricto de tiempo (3 horas sugeridas), se adoptaron los siguientes **supuestos operacionales razonables**:
1. **Consolidación Local Simulada:** El motor de base de datos (**PostgreSQL 17**) y el motor de respaldos (**pgBackRest**) operan de forma unificada utilizando almacenamiento local desacoplado lógicamente a través de volúmenes Docker. La separación física de infraestructura (nodos redundantes o almacenamiento de objetos inmutable) queda relegada a la fase final de despliegue en producción.
2. **Estrategia Remota Cloud-Native:** Atendiendo a restricciones de hardware del operador local, el diseño técnico se optimizó para ser portado de forma nativa a un entorno en la nube (**GitHub Codespaces / Instancia Linux**), garantizando que la solución sea completamente reproducible con un comando estándar de terminal.

## Mapa y Contenido de Archivos del Repositorio

Para garantizar la transparencia y el entendimiento del panel evaluador, a continuación se detallan los componentes técnicos incluidos en esta solución:

### 1. Orquestación y Despliegue
* **`docker-compose.yml`**: Define las variables de entorno de la base de datos (`POSTGRES_DB: conectarural`), expone el puerto estándar `5432` y monta los volúmenes compartidos de almacenamiento. Incorpora la directiva de inicio `postgres -c config_file=/etc/postgresql/postgresql.conf` para forzar al motor a leer nuestra gobernanza personalizada.

### 2. Infraestructura y Regulación de Datos (Carpeta `config/`)
* **`config/postgresql.conf`**: Archivo de configuración del motor principal. Modifica los parámetros críticos para la Transformación Digital: activa el archivado continuo (`archive_mode = on`), eleva el nivel de la bitácora a réplica (`wal_level = replica`) y delega el transporte físico de la bitácora directamente al comando interno de pgBackRest (`archive_command`).
* **`config/pgbackrest.conf`**: Configura el motor de respaldos. Define la estrofa (*stanza*) institucional de `conecta_rural`, direcciona la ruta física de las WALs de Postgres, establece políticas de retención completas (`repo1-retention-full=1`) y optimiza el uso de CPU mediante procesamiento paralelo acelerado.

### 3. Scripts de Simulación y Validación Operacional
* **`demo.sh`**: Script interactivo con **RUTs ficticios y secuenciales** según los lineamientos iniciales de la pauta. Muestra cronometradamente el flujo base del error del operador (RUTs 11.111.111-1 a 33.333.333-3), el aislamiento lógico y la restauración física.
* **`demo_2.sh`**: Script de contingencia masiva robustecido con **RUTs reales y algoritmos chilenos de validación** (ej. 15.674.321-K, 18.234.987-6). Diseñado para simular un comportamiento fidedigno de carga transaccional del subsidio en la Subsecretaría de Hacienda, demostrando estabilidad en el procesamiento de bloques WAL y un 0% de pérdida de registros.

## Respuestas Oficiales

### 1. ¿Si se pierde el disco de la base, cuántos datos se pierden (RPO)? ¿De qué depende?
* **Respuesta:** En la arquitectura implementada, la pérdida de datos potencial (RPO - Recovery Point Objective) se reduce a **cero registros comprometidos o al bloque de transacciones pendiente de archivado**.
* **Dependencia:** El RPO depende directamente del volumen transaccional del subsidio rural y del parámetro `archive_timeout` en `postgresql.conf`. Al ocurrir una pérdida catastrófica de hardware, únicamente se perderá el archivo WAL activo que esté siendo modificado y que no haya completado su transmisión (*archive-push*) hacia el repositorio seguro de pgBackRest.

### 2. ¿Cuánto tarda en volver a estar disponible (RTO)? ¿Cómo crece con el tamaño de la base de datos?
* **Respuesta:** En nuestro entorno controlado, el tiempo de recuperación y disponibilidad (RTO - Recovery Time Objective) es de **aproximadamente 12 a 15 segundos** (tiempo medido en la ejecución del script automatizado).
* **Escalamiento:** El RTO crece de forma **lineal con respecto al tamaño físico de los archivos de datos de la base** (duración del restore físico inicial) y de forma **proporcional a la cantidad de tráfico de transacciones lógicas acumuladas en las WAL** desde el último respaldo full o incremental. pgBackRest mitiga este impacto implementando algoritmos de compresión paralela (*Zstandard/LZ4*) y *Delta Restore*, que reescribe únicamente los bloques modificados reduciendo drásticamente las ventanas operacionales de caída.

### 3. ¿Qué pasó con lo escrito después del incidente y cómo lo trató?
* **Respuesta:** Las postulaciones legítimas recibidas posterior al error del operador (Punto E) se sometieron a una **Estrategia de Aislamiento Lógico Pre-Restauración**.
* **Tratamiento:** Al tratarse de una falla humana de software y no de un daño de hardware, la base de datos seguía operativa. Antes de proceder con el borrado físico que impone el Point-In-Time Recovery (PITR), se ejecutó un comando `COPY` lógico selectivo basado en marcas de tiempo (`fecha > TARGET_TIME`). Esto resguardó la data nueva en un archivo CSV independiente, permitiendo reimportarla de forma consolidada al finalizar la reconstrucción física sin alterar la continuidad de los postulantes de la crisis.

### 4. ¿Dónde deben vivir los respaldos? ¿Qué pasa si están en el mismo disco o servidor que la base?
* **Respuesta:** Los respaldos de pgBackRest **deben vivir de manera mandatoria en un nodo de infraestructura o servicio de almacenamiento de objetos aislado externamente** (como AWS S3, Google Cloud Storage o MinIO en la nube privada del Estado).
* **Impacto de Co-ubicación:** Si los respaldos residen en el mismo disco o servidor físico que aloja el motor principal de PostgreSQL, la solución presenta un **punto único de falla crítico**. Ante una falla eléctrica masiva, corrupción de sistema de archivos o ataque de Ransomware en el nodo, se perderían simultáneamente la base de datos operativa y los históricos de respaldo, anulando la resiliencia del servicio Conecta Rural.

### 5. ¿Cómo sabría, un martes cualquiera, que los respaldos de verdad sirven?
* **Respuesta:** Implementando un **Pipeline Automatizado de Convalida Técnica (Pruebas de Humo)** en un entorno alterno (Staging / QA).
* **Mecanismo:** No basta con verificar la salud sintáctica con un comando `pgbackrest check`. Cada martes, un proceso programado (*cronjob*) debe tomar un respaldo aleatorio del repositorio, restaurarlo de forma física y automatizada en un contenedor temporal, levantar el motor PostgreSQL y ejecutar una consulta de integridad (`SELECT COUNT(*)` en la tabla de postulaciones). Si el proceso concluye con éxito, se certifica la viabilidad real del desastre; si falla, levanta una alerta crítica automática al equipo de ingeniería de datos.

---

## Instrucciones para Ejecutar la Demostración

Para reproducir la simulación técnica del flujo del negocio y observar el comportamiento cronometrado de las fases analizadas, ejecute el siguiente comando en la consola:
# archivo demo.sh con RUTs ficticios 
# archivo demo_2.sh con RUTs reales

```bash
./demo.sh
```
