# Optimización de Utilización de Recursos – Café Web Application (AWS Lab)

## Overview
En este laboratorio se optimizan los recursos AWS utilizados por la aplicación web **Café**, con el objetivo de **reducir costos operativos** sin afectar la funcionalidad.

Las optimizaciones principales son:
- Eliminación de la base de datos local ya decomisionada.
- Reducción del tipo de instancia EC2 de **t3.small** a **t3.micro**.
- Estimación de ahorro de costos mediante **AWS Pricing Calculator**.

---

## Arquitectura

La topología de la aplicación Café incluye:
- Amazon VPC
- Instancia EC2 **CafeInstance**
- Amazon RDS (MariaDB)
- Amazon EBS
- CLI Host

<img width="1232" height="574" alt="image" src="https://github.com/user-attachments/assets/fcc35b33-356b-41f6-884f-6f7a4f426bf3" />

---

## Objetivos
- Optimizar una instancia Amazon EC2 para reducir costos.
- Eliminar componentes innecesarios tras una migración.
- Cambiar el tipo de instancia EC2.
- Estimar costos antes y después de la optimización.
- Calcular el ahorro mensual proyectado.

---

## Servicios Utilizados
- Amazon EC2
- Amazon EBS
- Amazon RDS (MariaDB)
- AWS CLI
- AWS Pricing Calculator

---

## Task 1: Optimizar la Instancia EC2

### 1.1 Configuración del AWS CLI
Se configuró el AWS CLI con credenciales del laboratorio.

```bash
aws configure
```

<img width="496" height="115" alt="image" src="https://github.com/user-attachments/assets/3a305d08-6026-436c-ad95-e35967c87fb2" />

---

### 1.2 Eliminación de la Base de Datos Local

La base de datos local MariaDB fue detenida y desinstalada:

```bash
sudo systemctl stop mariadb
sudo yum -y remove mariadb-server
```

<img width="1096" height="994" alt="image" src="https://github.com/user-attachments/assets/cb45e4a1-42ce-463a-996c-fc3b69cf29ab" />

---

### 1.3 Identificación de la Instancia EC2

Desde el **CLI Host**, se obtuvo el Instance ID:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=CafeInstance" --query "Reservations[*].Instances[*].InstanceId"
```

<img width="477" height="153" alt="image" src="https://github.com/user-attachments/assets/2ea7fac6-cde4-44bc-b534-2607a4051047" />

---

### 1.4 Redimensionamiento de la Instancia

Se detuvo la instancia, se modificó el tipo y se inició nuevamente:

```bash
aws ec2 stop-instances --instance-ids <instance-id>

aws ec2 modify-instance-attribute --instance-id <instance-id> --instance-type "{\"Value\": \"t3.micro\"}"

aws ec2 start-instances --instance-ids <instance-id>
```

<img width="733" height="366" alt="image" src="https://github.com/user-attachments/assets/ea8219c6-4438-4075-b1fc-f5253100f870" />
<img width="1574" height="150" alt="image" src="https://github.com/user-attachments/assets/2e49af73-4175-4895-bfd5-fc1872e73280" />

---

### 1.5 Verificación de Funcionamiento

Se verificó el estado y se accedió al sitio web:

```bash
aws ec2 describe-instances --instance-ids <instance-id> --query "Reservations[*].Instances[*].[InstanceType,PublicDnsName,PublicIpAddress,State.Name]"
```

<img width="1893" height="172" alt="image" src="https://github.com/user-attachments/assets/4189adf9-e825-4758-9221-33a9ec06de93" />

URL probada:
```
http://<Public-DNS>/cafe
```

<img width="1084" height="961" alt="image" src="https://github.com/user-attachments/assets/c7a2b303-2485-439f-806e-d454800158eb" />

---

## Task 2: Estimación de Costos con AWS Pricing Calculator

### 2.0 Costos Antes de la Optimización

Configuración:
- EC2: t3.small
- EBS: 40 GB gp2
- RDS: db.t3.micro (20 GB)

<img width="1864" height="751" alt="image" src="https://github.com/user-attachments/assets/170523d5-b898-47b0-b0d2-209efea1bfbd" />

**Costo mensual estimado:** $74.04

---

### 2.1 Costos Después de la Optimización

Cambios:
- EC2: t3.micro
- EBS: 20 GB gp2

<img width="1863" height="756" alt="image" src="https://github.com/user-attachments/assets/c7114245-bd14-476d-b40b-137d907a2ae4" />

**Costo mensual estimado:** $64.45

---

### 2.2 Ahorro Proyectado

| Estado | Costo Mensual |
|------|---------------|
| Antes | $74.04 |
| Después | $64.45 |
| **Ahorro** | **$9.59** |

---

## Resultados
- Reducción exitosa del tamaño de la instancia.
- Eliminación de recursos innecesarios.
- Ahorro mensual demostrable.
- Aplicación de buenas prácticas de optimización de costos.

---

## Qué Haría Distinto en Producción
- Automatizar right-sizing con **AWS Compute Optimizer**
- Usar **Elastic IP** para evitar cambios de IP
- Implementar **Infrastructure as Code**
- Monitorear costos con **AWS Budgets**
- Aplicar políticas de apagado automático

---

## Conclusión
Este laboratorio demuestra cómo pequeñas optimizaciones técnicas pueden generar **impacto financiero positivo**, reforzando la importancia del **cost optimization** como pilar del AWS Well-Architected Framework.
