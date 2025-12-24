# Gestión de Recursos con Etiquetas en Amazon EC2 (AWS Lab)

## Descripción general
En este laboratorio se utiliza el **sistema de etiquetas (tags) de AWS** para **identificar, administrar y automatizar acciones** sobre recursos de **Amazon EC2**.

El laboratorio se divide en dos partes:
- **Task:** uso de AWS CLI para consultar y modificar recursos según sus etiquetas.
- **Challenge:** diseño de un enfoque automatizado para **terminar instancias no conformes** (sin etiquetas obligatorias).

Este escenario refleja prácticas reales de **gobernanza, control de costos y seguridad** en entornos cloud.

<img width="1130" height="426" alt="image" src="https://github.com/user-attachments/assets/c311f291-f30c-4caa-a984-a8f168ada066" />

---

## Arquitectura
La arquitectura del laboratorio incluye:
- Amazon VPC (Lab VPC)
- Subnet pública y privada
- Instancia **CommandHost** (AWS CLI preconfigurado)
- Múltiples instancias EC2 con etiquetas personalizadas

> 📸 **Captura sugerida:** Vista general de instancias EC2 con sus tags visibles en la consola.

---

## Objetivos
- Aplicar y modificar etiquetas en recursos EC2
- Buscar recursos utilizando tags
- Usar AWS CLI con filtros y JMESPath
- Automatizar acciones (stop/start/terminate) basadas en etiquetas
- Implementar un enfoque *tag-or-terminate*

---

## Servicios utilizados
- Amazon EC2
- AWS CLI
- AWS SDK for PHP
- Amazon VPC

---

## Implementación del laboratorio

### 1. Acceso al Command Host
Se estableció conexión SSH con la instancia **CommandHost**, desde donde se ejecutaron todos los comandos AWS CLI.

> 📸 **Captura sugerida:** Terminal conectado al CommandHost.

---

### 2. Identificación de instancias mediante tags
Se usó AWS CLI para identificar instancias EC2 pertenecientes al proyecto **ERPSystem**.

```bash
aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem"
```

> 📸 **Captura sugerida:** Salida del comando mostrando instancias filtradas por tag.

---

### 3. Uso de JMESPath para salida estructurada
Se aplicaron consultas `--query` para obtener resultados legibles y específicos.

```bash
aws ec2 describe-instances  --filter "Name=tag:Project,Values=ERPSystem"  --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone}'
```

> 📸 **Captura sugerida:** Resultado formateado con ID y AZ.

---

### 4. Visualización de etiquetas personalizadas
Se incluyeron los valores de **Project**, **Environment** y **Version** en la salida.

> 📸 **Captura sugerida:** Resultado CLI mostrando Project, Environment y Version.

---

### 5. Modificación masiva de tags (Version)
Se ejecutó un script Bash para actualizar la etiqueta **Version** de instancias *development*.

```bash
./change-resource-tags.sh
```

> 📸 **Captura sugerida:** Ejecución exitosa del script y validación posterior.

---

## Automatización basada en etiquetas

### 6. Detención de instancias por tag
Se utilizó el script **stopinator.php** para detener instancias de desarrollo.

```bash
./stopinator.php -t"Project=ERPSystem;Environment=development"
```

> 📸 **Captura sugerida:**  
> - Salida del script identificando instancias  
> - Instancias en estado *stopping* en la consola EC2

---

### 7. Inicio de instancias por tag
Las mismas instancias fueron iniciadas nuevamente.

```bash
./stopinator.php -t"Project=ERPSystem;Environment=development" -s
```

> 📸 **Captura sugerida:** Instancias regresando a estado *running*.

---

## Challenge: Tag-Or-Terminate

### 8. Identificación de instancias no conformes
Se simularon instancias sin la etiqueta **Environment**.

> 📸 **Captura sugerida:** Instancias sin tag Environment en la consola.

---

### 9. Terminación automática de instancias
Se ejecutó el script **terminate-instances.php** para eliminar instancias no conformes.

```bash
./terminate-instances.php -region <region> -subnetid <subnet-id>
```

> 📸 **Captura sugerida:**  
> - Salida del script terminando instancias  
> - Confirmación en consola EC2

---

## Resultados
- Recursos administrados eficientemente mediante etiquetas
- Automatización segura de operaciones EC2
- Eliminación de recursos no conformes
- Mejora en gobernanza y control operativo

---

## Qué haría distinto en producción
- Usar **AWS Config Rules** para compliance automático
- Implementar **IAM con mínimo privilegio**
- Automatizar con **Lambda + EventBridge**
- Bloquear creación de recursos sin tags obligatorios
- Integrar con políticas de costos

---

## Conclusión
Este laboratorio demuestra cómo las **etiquetas son fundamentales para la automatización, seguridad y gobernanza** de recursos en AWS, permitiendo operaciones eficientes y escalables.
