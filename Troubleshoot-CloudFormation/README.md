# Troubleshooting de AWS CloudFormation – Infraestructura como Código (IaC)

## Overview
Este laboratorio se centra en el **troubleshooting de despliegues con AWS CloudFormation** y refuerza conceptos clave de **Infraestructura como Código (IaC)**.  
La actividad simula escenarios reales donde los stacks de CloudFormation fallan durante la creación, actualización o eliminación, requiriendo un proceso sistemático de análisis utilizando **AWS CLI**, **JMESPath** y **análisis de logs en instancias EC2**.

El laboratorio se presenta como una prueba de concepto (POC) para la aplicación **Café**, permitiendo comprender cómo CloudFormation facilita despliegues confiables, detección de drift y gestión de infraestructura AWS a escala.

---

## Arquitectura

<img width="921" height="410" alt="image" src="https://github.com/user-attachments/assets/065a7cef-b5b3-46e0-af0c-39b19028aaae" />

La arquitectura utilizada a lo largo del laboratorio incluye:

- Amazon VPC personalizada (**VPC2**)
- Subred pública
- Instancias Amazon EC2:
  - **CLI Host** (administración y troubleshooting)
  - **Web Server** (creada por CloudFormation)
- Bucket de Amazon S3 (creado por el stack)
- Stack de AWS CloudFormation y recursos asociados
- Security Groups

Esta arquitectura se utiliza en las tareas 1, 2 y 3 para demostrar despliegues, detección de drift y desafíos al eliminar stacks.

---

## Objetivos
- Practicar consultas sobre datos JSON utilizando **JMESPath**.
- Diagnosticar fallos en despliegues de stacks de AWS CloudFormation.
- Analizar logs de **cloud-init** en instancias EC2 para identificar causas raíz.
- Detectar configuraciones fuera de sincronía (drift).
- Resolver eliminaciones fallidas de stacks preservando recursos críticos.
- Reforzar buenas prácticas de **Infraestructura como Código**.

---

## Servicios Utilizados
- AWS CloudFormation
- Amazon EC2
- Amazon VPC
- Amazon S3
- AWS CLI
- Linux (Amazon Linux)
- JMESPath

---
## Task 1: Troubleshooting en la Creación de Stacks con CloudFormation

### 1.1 Configuración del AWS CLI
El AWS CLI fue configurado con credenciales temporales del laboratorio y la región detectada mediante el servicio de metadata de EC2.

<img width="971" height="215" alt="image" src="https://github.com/user-attachments/assets/02f8a1ff-6b02-4a5a-a976-bba0c823143b" />

### 1.2 Fallo Inicial en la Creación del Stack
El primer intento de creación del stack falló y entró en estado **ROLLBACK_COMPLETE**.

<img width="1229" height="592" alt="image" src="https://github.com/user-attachments/assets/5c8d1e0a-d825-4b25-85e4-a9999bd000ab" />

<img width="1899" height="325" alt="image" src="https://github.com/user-attachments/assets/08af2e5f-e8fe-4785-bfce-659bf367bec0" />

El análisis de eventos reveló:
- Timeout en un **WaitCondition**
- Fallo en el script de userdata de la instancia EC2

### 1.3 Prevención del Rollback para Análisis
El stack fue recreado usando la opción `--on-failure DO_NOTHING`, lo que permitió inspeccionar los logs de la instancia EC2.

<img width="1031" height="176" alt="image" src="https://github.com/user-attachments/assets/88afdabd-3d43-4b43-9637-e4f3ea1698a1" />

<img width="1899" height="197" alt="image" src="https://github.com/user-attachments/assets/73480fd6-5a73-43a2-997c-2acacced6c75" />

<img width="1907" height="195" alt="image" src="https://github.com/user-attachments/assets/a9d33798-10c9-48b9-b4ec-bb0bc2b06514" />

El análisis de logs confirmó:
- Uso de un paquete inexistente (`http`).
- Terminación inmediata del script debido al flag `-e`.
- El WaitCondition nunca recibió la señal de éxito.

### 1.4 Corrección del Template y Despliegue Exitoso
El template de CloudFormation fue corregido reemplazando `http` por `httpd`.

<img width="804" height="182" alt="image" src="https://github.com/user-attachments/assets/295008bf-5e7a-4d8e-b50e-533fe64834fd" />

<img width="1340" height="344" alt="image" src="https://github.com/user-attachments/assets/d1e57fab-6999-4ff3-b16e-f55e6e0b77d1" />

<img width="546" height="105" alt="image" src="https://github.com/user-attachments/assets/42e9561f-587f-421c-9e49-2ca74ecda41d" />

Luego de recrear el stack:
- Todos los recursos se crearon correctamente.
- El stack alcanzó el estado **CREATE_COMPLETE**.
- El servidor web respondió correctamente.

---

## Task 2: Cambios Manuales y Detección de Drift

### 2.1 Modificaciones Manuales
Se modificó manualmente un Security Group para restringir el acceso SSH a una IP específica.

<img width="1653" height="275" alt="image" src="https://github.com/user-attachments/assets/9971eb7e-c40a-4b13-91b0-4a7619d50059" />

Adicionalmente, se cargó un objeto en el bucket S3 creado por el stack.

<img width="565" height="249" alt="image" src="https://github.com/user-attachments/assets/b1088fe2-7385-4956-b175-8f050cf4e4e2" />

### 2.2 Detección de Drift
Se ejecutó la detección de drift, obteniendo como resultado:
- Security Group en estado **MODIFIED**
- Bucket S3 en estado **IN_SYNC**

<img width="1043" height="268" alt="image" src="https://github.com/user-attachments/assets/89e131f7-6c15-46db-979b-1c1ddcde261f" />

<img width="1903" height="518" alt="image" src="https://github.com/user-attachments/assets/d2436125-495f-4ca2-a85a-994615f037b5" />

<img width="946" height="381" alt="image" src="https://github.com/user-attachments/assets/9a2634c3-4926-4302-8464-7452ebdeb933" />

Esto demuestra que CloudFormation detecta cambios de configuración, pero no considera el contenido de un bucket como drift.

---

## Task 3: Fallo en la Eliminación del Stack
<img width="1231" height="280" alt="image" src="https://github.com/user-attachments/assets/cc32564f-46a8-4861-9866-8302dde595fd" />

<img width="601" height="306" alt="image" src="https://github.com/user-attachments/assets/ad7bf03e-c594-4c83-aa16-c4539b0025f3" />

Al intentar eliminar el stack, este quedó en estado **DELETE_FAILED**.

Causa:
- CloudFormation no elimina buckets S3 que contienen objetos.

---

## Desafío: Conservar el Bucket S3 y Eliminar el Stack
<img width="1050" height="326" alt="image" src="https://github.com/user-attachments/assets/cec40414-23aa-416c-acab-61801d2a0bdd" />

Utilizando AWS CLI:
- Se identificó el LogicalResourceId del bucket.
- Se eliminó el stack usando la opción `--retain-resources`.

Resultado:
- Eliminación exitosa del stack.
- Conservación del bucket S3 y su contenido.

---

## Resultados
- Identificación y resolución de fallos en despliegues CloudFormation.
- Análisis de logs EC2 para encontrar causas raíz.
- Uso efectivo de JMESPath en AWS CLI.
- Detección y análisis de drift.
- Eliminación de stacks preservando recursos críticos.
- Aplicación de prácticas reales de troubleshooting en IaC.

---

## Qué Haría Distinto en Producción
- Validación y linting de templates (cfn-lint).
- Uso de Change Sets antes de aplicar cambios.
- Políticas de stack para proteger recursos críticos.
- Centralización de logs y monitoreo.
- Control estricto para evitar cambios fuera de IaC.

---

## Conclusión
Este laboratorio demuestra la importancia de un **troubleshooting metódico** al trabajar con **Infraestructura como Código**.  
El uso combinado de AWS CloudFormation, AWS CLI, JMESPath y análisis de logs permite construir entornos **confiables, auditables y listos para producción**, alineados con las mejores prácticas de AWS.
