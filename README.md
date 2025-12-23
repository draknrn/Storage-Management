# Storage Management – AWS Lab

##  Descripción general
En este laboratorio trabajé con **Amazon EBS y Amazon S3** para aprender a **gestionar respaldos de forma automatizada** y a **sincronizar archivos de manera segura**, usando principalmente **AWS CLI, Python y servicios core de AWS**.

El objetivo principal fue:
- Crear snapshots de volúmenes EBS
- Automatizar su creación y limpieza
- Mantener solo los respaldos más recientes
- Sincronizar archivos desde una instancia EC2 hacia Amazon S3
- Recuperar archivos eliminados usando versionado

Este laboratorio simula un escenario real de administración de infraestructura en la nube.

---

##  Arquitectura del laboratorio

El entorno estaba compuesto por:
- Una **VPC** con una subnet pública
- Dos instancias EC2:
- **Command Host**: administración y ejecución de comandos AWS CLI
- **Processor**: instancia con volumen EBS y archivos de prueba
- Un **volumen EBS** adjunto a Processor
- Un **bucket Amazon S3** inicialmente sin versionado, el cual fue habilitado posteriormente mediante AWS CLI para permitir recuperación de archivos

### Diagrama del laboratorio o vista general de EC2 + S3

<img width="1538" height="970" alt="image" src="https://github.com/user-attachments/assets/eb31cec0-d245-4070-8bb5-b2d703447ab1" />

---

##  Objetivos del laboratorio

- Crear snapshots manuales y automáticos de un volumen EBS
- Programar tareas usando `cron`
- Automatizar la limpieza de snapshots con Python
- Sincronizar archivos desde EC2 hacia S3
- Recuperar archivos eliminados gracias al versionado de S3

---

##  Tecnologías y servicios utilizados

- Amazon EC2
- Amazon EBS
- Amazon S3
- AWS CLI
- IAM Roles
- Python 3
- Cron (Linux Scheduler)

---

##  Desarrollo del laboratorio

###  1. Creación del bucket S3
Se creó un bucket S3 que actuaría como destino para la sincronización de archivos desde la instancia EC2.

### Creación del bucket en la consola S3

<img width="1373" height="649" alt="image" src="https://github.com/user-attachments/assets/ae5dc3dc-7cb9-4b42-896d-bb13489751d3" />

---

###  2. Asignación de permisos (IAM Role)
Se asignó un **IAM Role** a la instancia **Processor** para permitirle interactuar con Amazon S3 y otros servicios sin usar credenciales hardcodeadas.

### EC2 → Modify IAM Role

<img width="1441" height="518" alt="image" src="https://github.com/user-attachments/assets/525e7168-a782-4ad1-9b76-382c0ae07ed9" />

---

###  3. Creación de snapshot inicial del volumen EBS
Desde la instancia **Command Host**, utilicé AWS CLI para:
- Identificar el volumen EBS asociado
- Detener la instancia Processor
- Crear un snapshot inicial
- Reiniciar la instancia

Esto garantiza consistencia del respaldo.

### Snapshot creado en la consola EC2

<img width="761" height="224" alt="image" src="https://github.com/user-attachments/assets/4e834c37-e54f-4ff2-966d-fee8512250b6" />

---

###  4. Automatización de snapshots con cron
Configuré una tarea programada (`cron`) que crea snapshots **cada minuto**, solo con fines demostrativos para el laboratorio.

Esto simula un sistema automático de respaldos periódicos.

<img width="1204" height="599" alt="image" src="https://github.com/user-attachments/assets/84a23ed8-c8d1-485d-9e6b-add540116bf4" />
<img width="1301" height="137" alt="image" src="https://github.com/user-attachments/assets/e9daa2ae-f343-444e-b51b-139c516e7c4c" />

---

###  5. Limpieza automática de snapshots antiguos (Python)
Ejecuté un script en Python que:
- Lista todos los snapshots del volumen
- Los ordena por fecha
- Elimina todos excepto los **2 más recientes**

Esto es clave para:
- Controlar costos
- Mantener solo respaldos relevantes

<img width="1300" height="171" alt="image" src="https://github.com/user-attachments/assets/dac706c8-24a8-4cdf-b682-c8ffc80dd580" />

---

##  Desafío: Sincronización con Amazon S3

###  6. Activación de versionado en S3
Habilité **versionado** en el bucket S3 para poder recuperar archivos eliminados accidentalmente.

### Versioning habilitado en el bucket

<img width="1109" height="86" alt="image" src="https://github.com/user-attachments/assets/03889130-e170-4d1a-bdef-84b470b8a93c" />

---

###  7. Sincronización de archivos con `aws s3 sync`
Sincronicé una carpeta local desde la instancia EC2 hacia S3 usando:

```bash
aws s3 sync files s3://mi-bucket/files/
```
### Salida del comando sync

<img width="536" height="52" alt="image" src="https://github.com/user-attachments/assets/2b519dce-d014-4d34-8c54-5c3ea4c13195" />

Esto copia y mantiene sincronizados los archivos entre EC2 y S3.

### 8. Eliminación y recuperación de archivos
- Eliminé un archivo localmente
- Sincronicé usando --delete
- Recuperé el archivo desde una versión anterior en S3
Este paso demuestra recuperación ante errores humanos, un escenario muy común en producción.

### Archivo eliminado
<img width="706" height="241" alt="image" src="https://github.com/user-attachments/assets/1786bb84-237a-4a15-a895-034b749f7fc2" />

### Versiones del objeto S3
<img width="942" height="578" alt="image" src="https://github.com/user-attachments/assets/113ba89f-9a3c-46c3-98fb-5c30900d7e07" />

### Archivo restaurado
<img width="1384" height="256" alt="image" src="https://github.com/user-attachments/assets/d9762a42-f78c-4ba5-beb2-cc13ce3879d9" />

### Sincronización con el bucket de S3
<img width="626" height="102" alt="image" src="https://github.com/user-attachments/assets/e63f9d7f-671d-48d2-a9d6-75eba35e17d6" />

## Resultados
- Snapshots automatizados
- Limpieza automática de respaldos
- Sincronización segura con S3
- Recuperación de archivos eliminados
- Uso de buenas prácticas de seguridad (IAM Roles)

## Aprendizajes clave
- La automatización reduce errores y costos
- EBS snapshots son esenciales para respaldos de EC2
- S3 + versionado es una excelente capa de protección de datos
- AWS CLI permite administrar infraestructura sin depender de la consola
- Python es ideal para tareas repetitivas de administración cloud

## Conclusión
Este laboratorio demuestra habilidades prácticas en gestión de almacenamiento en AWS, automatización y recuperación de datos, alineadas con escenarios reales de trabajo en la nube.
