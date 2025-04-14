# Monitoreo de un Clúster AWS EMR con Prometheus y Grafana

Este proyecto presenta la implementación de un sistema de monitoreo completo para un clúster AWS EMR, utilizando herramientas open-source como **Prometheus**, **Grafana** y **JMX Exporter**. La solución permite observar en tiempo real el estado del clúster, métricas del sistema y uso de recursos, facilitando la administración y mantenimiento de la infraestructura.

## 🧠 Objetivo

- Configurar un clúster EMR en AWS.
- Exponer métricas del clúster con JMX Exporter.
- Recopilar y almacenar métricas con Prometheus.
- Visualizar métricas mediante dashboards en Grafana.

---

## ⚙️ Configuración del Clúster EMR

1. **Creación del clúster EMR:**
   - Nombre: `EMR-Monitoring-iabdXX`
   - Aplicaciones: Hadoop, Spark, Hive.
   - Tipo de instancias: 1 nodo maestro, 2 nodos core.
   - Clave SSH: por defecto en el laboratorio.
2. **Exposición del puerto 7070** en el grupo de seguridad.
3. **Conexión por SSH** al nodo maestro del clúster.

---

## 📦 JMX Exporter

### ¿Qué es JMX?

JMX (Java Management Extensions) permite la monitorización y gestión de aplicaciones Java. A través de JMX Exporter, Prometheus puede recolectar métricas de los servicios Hadoop y Spark, sin necesidad de modificar su código fuente.

### Pasos de configuración:

1. **Instalación del agente:**
   ```bash
   wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.16.1/jmx_prometheus_javaagent-0.16.1.jar
   ```

2. **Creación del archivo `config.yml`:**
   ```yaml
   lowercaseOutputName: true
   lowercaseOutputLabelNames: true
   rules:
     - pattern: ".*"
   ```

3. **Modificar configuración de NameNode:**
   Editar `/etc/hadoop/conf/hadoop-env.sh` y añadir:
   ```bash
   export HADOOP_NAMENODE_OPTS="-javaagent:/home/hadoop/jmx_prometheus_javaagent-0.16.1.jar=12345:/home/hadoop/config.yml $HADOOP_NAMENODE_OPTS"
   ```

4. **Reiniciar NameNode:**
   ```bash
   sudo systemctl restart hadoop-hdfs-namenode
   ```

---

## 📈 Prometheus y Grafana

### Prometheus

1. **Descarga e instalación:**
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.30.3/prometheus-2.30.3.linux-amd64.tar.gz
   tar -xzf prometheus-2.30.3.linux-amd64.tar.gz
   cd prometheus-2.30.3.linux-amd64
   ```

2. **Configuración (`prometheus.yml`):**
   ```yaml
   scrape_configs:
     - job_name: 'emr-namenode'
       static_configs:
         - targets: ['<ip-nodo-maestro>:12345']
   ```

3. **Ejecutar Prometheus:**
   ```bash
   ./prometheus --config.file=prometheus.yml
   ```

### Grafana

1. **Instalación:**
   ```bash
   sudo apt-get install -y apt-transport-https software-properties-common wget
   wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
   echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
   sudo apt-get update
   sudo apt-get install grafana
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```

2. **Acceder a Grafana:**
   Navegar a `http://<ip-ec2>:3000` y configurar Prometheus como fuente de datos (`http://localhost:9090`).

---

## 📊 Dashboards en Grafana

Se crearon paneles para visualizar las siguientes métricas:

- **Uso de CPU**
- **Uso de RAM**
- **Espacio utilizado en HDFS**
- **Estado del NameNode**

Cada panel fue configurado a partir de las métricas expuestas por JMX Exporter.

---

## 🤔 Reflexiones

**1. ¿Qué métricas son críticas en EMR?**

- Uso de CPU/RAM: para detectar sobrecargas.
- Espacio HDFS: prevenir fallos por capacidad.
- Estado del NameNode/DataNodes: componentes clave del clúster.

**2. ¿Cómo mejorar JMX Exporter?**

- Especificar reglas por componente.
- Añadir etiquetas.
- Filtrar duplicados y ruido.
- Separar configuraciones por servicio.

**3. ¿Por qué usar Prometheus y Grafana?**

- Open source y gratuitos.
- Integración sencilla con JMX.
- Alta personalización.
- Gran rendimiento en entornos distribuidos.

---
## 👤 Autor

**Miguel Sedano Izurieta**  
Big Data Aplicado - IABD  
2025
