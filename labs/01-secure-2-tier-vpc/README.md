# Arquitectura Segura de 2 Capas con AWS VPC y Application Load Balancer (ALB)

## Descripción del Proyecto
Diseño e implementación de una infraestructura de red segura y altamente disponible (High Availability) en AWS utilizando un enfoque de **DMZ** y **Capa Privada Aislada**. 
Este laboratorio simula un entorno real alineado con los requerimientos del examen **AWS Certified Solutions Architect - Associate (SAA-C03)**.

El objetivo principal es aislar los servidores de cómputo en subredes privadas sin direcciones IP públicas, permitiendo el acceso web de los usuarios exclusivamente a través de un balanceador de carga público.

## Diagrama de Arquitectura (Resource Map)
![VPC Resource Map](./screenshots/01-vpc-resource-map.png)

## Especificaciones Técnicas y de Red

### 1. Direccionamiento IP (VPC & Subnets)
* **VPC CIDR Block:** `10.0.0.0/16`
* **Subredes Públicas (Capa Front-End / DMZ):**
  * `Pub-Subnet-A` (`10.0.1.0/24`) en la AZ `us-east-2a`
  * `Pub-Subnet-B` (`10.0.2.0/24`) en la AZ `us-east-2b`
* **Subredes Privadas (Capa de Aplicación / LAN Interna):**
  * `Priv-Subnet-A` (`10.0.11.0/24`) en la AZ `us-east-2a`
  * `Priv-Subnet-B` (`10.0.12.0/24`) en la AZ `us-east-2b`

### 2. Tabla de Enrutamiento y Conectividad Perimeter
* **Tablas Públicas:** Asociación con `Pub-Subnet-A` y `Pub-Subnet-B`. Ruta por defecto: `0.0.0.0/0 -> Internet Gateway (IGW)`.
* **Tablas Privadas:** Asociación con `Priv-Subnet-A` y `Priv-Subnet-B`. Completamente aisladas (sin ruta `0.0.0.0/0` en estado final).

### 3. Seguridad de Capa de Host (Security Groups - Stateful Firewalls)
* **ALB-SG (Balanceador):**
  * Entrada (Inbound): `TCP 80` (HTTP) desde cualquier origen (`0.0.0.0/0`).
* **EC2-SG (Servidores Web):**
  * Entrada (Inbound): `TCP 80` (HTTP) encadenado únicamente al origen lógico del `ALB-SG`. Bloquea cualquier petición directa de internet.

---

## Conceptos Clave Aprendidos (Enfoque SAA-C03)

* **Alta Disponibilidad:** Despliegue multi-zona utilizando dos Zonas de Disponibilidad (AZs) para evitar puntos únicos de falla.
* **Seguridad Stateful:** Comprensión de cómo los *Security Groups* permiten el tráfico de retorno de forma automática sin requerir reglas de salida explícitas o rutas de internet directas para el cliente.
* **Troubleshooting de Red en AWS (Error 502 Bad Gateway):** Diagnóstico y resolución de problemas de inicialización de software (*User Data*) causados por la falta de rutas de salida o NAT en subredes privadas aisladas.

## Evidencias de Funcionamiento

### Servidores EC2 Aislados (Sin IP Pública)
![EC2 Details](./screenshots/02-ec2-isolated.png)

### Balanceo Round-Robin Exitoso (Output del Navegador)
Al refrescar el DNS del ALB, se comprueba la distribución de carga entre nodos internos:
* Respuesta Nodo A: `Hola desde mi Red Privada. Mi IP interna es: 10.0.11.X`
* Respuesta Nodo B: `Hola desde mi Red Privada. Mi IP interna es: 10.0.12.X`

### Estado Saludable en Target Group
![Target Group Healthy](./screenshots/03-alb-healthy.png)
