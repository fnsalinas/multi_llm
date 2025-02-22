# 🚀 Configuración de Instancia AWS EC2 para DeepSeek
## Proyecto: Prueba de Concepto con DeepSeek y AWS EC2

---

## **1️⃣ Resumen del Proyecto**
Este documento describe la configuración de una instancia **AWS EC2** optimizada para ejecutar el modelo **DeepSeek LLM** en la imagen **Deep Learning Base OSS Nvidia Driver GPU AMI (Ubuntu 22.04)**. Se han seleccionado opciones equilibradas entre **rendimiento y costo** para una **prueba de concepto**.

---

## **2️⃣ Configuración de la Instancia**
### 📌 **Nombre de la instancia**
- `multi_llm_poc1`

### 📌 **AMI seleccionada**
- **Deep Learning Base OSS Nvidia Driver GPU AMI (Ubuntu 22.04)**
- **ID AMI:** `ami-095694d8118689b4` (64 bits x86)
- **Fuente:** AWS Marketplace

### 📌 **Tipo de instancia**
- **Instancia:** `c6i.large`
  - **2 vCPUs**
  - **8 GB RAM**
  - **Optimizado para computación**
  - **Costo aprox.:** `$0.177 USD/hora` (on-demand)

---

## **3️⃣ Configuración de Seguridad y Red**
### 📌 **Claves SSH**
- **Nombre de clave:** `multi_llm_poc`
- **Método:** Autenticación con clave pública

### 📌 **Configuración de Red**
- **VPC:** `Default`
- **Subred:** `Predeterminada (asignación automática de IP)`
- **Grupo de seguridad:** 
  - Permitir **SSH (Puerto 22) desde cualquier IP** `0.0.0.0/0`
  - Permitir **HTTPS (Puerto 443)**

> ⚠ **Importante:** Se recomienda restringir el acceso SSH solo a direcciones IP de confianza.

---

## **4️⃣ Configuración de Almacenamiento**
- **Volumen raíz:** `100 GiB (gp3)`
  - **IOPS:** `3000`
  - **Throughput:** `125 MB/s`
- **Cifrado:** No habilitado (opcional para producción)

---

## **5️⃣ Configuración Avanzada**
### 📌 **Configuración de CPU**
- **Uso de GPU:** `Habilitado`
- **Uso de CPU optimizado:** `Automático`

### 📌 **Configuración de Metadatos**
- **Protección IMDSv2:** `Habilitado`
- **Límite de llamadas:** `Predeterminado`

---

## **6️⃣ Script de Inicialización (User Data)**
El siguiente **script de User Data** se ejecutará al iniciar la instancia, asegurando la configuración automática de entorno.

```bash
#!/bin/bash

# Redirigir salida estándar y errores a un log
exec > /var/log/user-data.log 2>&1

echo "----------------------------"
echo "🚀 INICIO DE CONFIGURACIÓN 🚀"
echo "----------------------------"

# Variables del entorno
DEEPSEEK_PATH="/opt/deepseek"
DOCKER_COMPOSE_VERSION="1.29.2"

echo "📌 Actualizando el sistema..."
apt update && apt upgrade -y

echo "📌 Instalando paquetes esenciales..."
apt install -y python3 python3-pip docker.io unzip wget git curl nvidia-utils-535

# Verificar si Docker ya está instalado
if ! command -v docker &> /dev/null; then
    echo "📌 Instalando Docker..."
    apt install -y docker.io
    systemctl enable docker
    systemctl start docker
    usermod -aG docker ubuntu
else
    echo "✅ Docker ya está instalado."
fi

# Verificar si NVIDIA está funcionando correctamente
echo "📌 Verificando estado de la GPU..."
if nvidia-smi &> /dev/null; then
    echo "✅ NVIDIA detectada correctamente."
else
    echo "❌ ERROR: No se detectó NVIDIA. Verifica los drivers."
    exit 1
fi

# Instalar NVIDIA-Docker
echo "📌 Instalando NVIDIA-Docker..."
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && \
wget -qO - https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add - && \
wget -qO - https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list && \
apt update && apt install -y nvidia-docker2 && systemctl restart docker

# Verificar que NVIDIA-Docker funcione correctamente
if docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi &> /dev/null; then
    echo "✅ NVIDIA-Docker funcionando correctamente."
else
    echo "❌ ERROR: NVIDIA-Docker no funciona. Revisa la configuración."
    exit 1
fi

# Instalar Docker Compose
echo "📌 Instalando Docker Compose..."
if ! command -v docker-compose &> /dev/null; then
    curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    echo "✅ Docker Compose instalado correctamente."
else
    echo "✅ Docker Compose ya está instalado."
fi

# Descargar e instalar DeepSeek
if [ ! -d "$DEEPSEEK_PATH" ]; then
    echo "📌 Descargando DeepSeek..."
    git clone https://github.com/DeepSeek-AI/DeepSeek-LLM.git "$DEEPSEEK_PATH"
    cd "$DEEPSEEK_PATH" || exit 1
    pip3 install -r requirements.txt
    echo "✅ DeepSeek instalado en $DEEPSEEK_PATH."
else
    echo "✅ DeepSeek ya está instalado."
fi

# Configurar acceso SSH y logs básicos
echo "📌 Configurando SSH..."
systemctl enable ssh
systemctl start ssh

echo "📌 Creando log de inicio..."
echo "Instancia inicializada correctamente" > /var/log/deepseek-setup.log

echo "----------------------------"
echo "🎯 CONFIGURACIÓN FINALIZADA 🎯"
echo "----------------------------"
```

---

## **7️⃣ Resumen Final**
- **Instancia:** `g4dn.xlarge`
- **GPU:** `NVIDIA T4 (16GB VRAM)`
- **Almacenamiento:** `100 GiB (gp3)`
- **Seguridad:** `SSH habilitado, HTTPS habilitado`
- **Configuración automática:** `User Data con Docker, NVIDIA y DeepSeek`
- **Costo estimado:** `$0.526 USD/hora (On-Demand)`

---

## **8️⃣ Recomendaciones Adicionales**
✅ **Usar instancias Spot** para reducir costos hasta un **70%**.  
✅ **Monitorear GPU y CPU** con `nvidia-smi` y `htop`.  
✅ **Restringir el acceso SSH** a una IP específica para mayor seguridad.  

---

## **9️⃣ Comandos Útiles**
Una vez desplegada la instancia, usa los siguientes comandos para verificar la configuración:

### **📌 Verificar estado de la GPU**
```bash
nvidia-smi

### **📌 Verificar que Docker está corriendo**
```bash
systemctl status docker
```

### **📌 Revisar logs del script de inicialización (User Data)**
```bash
cat /var/log/user-data.log
```

### **📌 Revisar logs específicos de la configuración de DeepSeek**
```bash
cat /var/log/deepseek-setup.log
```

### **📌 Conectar a la instancia vía SSH**
```bash
ssh -i multi_llm_poc.pem ubuntu@<IP_PUBLICA>
```

** ⚠ Nota: Reemplaza <IP_PUBLICA> con la dirección IP de la instancia en AWS.**

---

## ** Proximos Pasos**

1. ✅ Configurar entorno virtual para DeepSeek
```bash
cd /opt/deepseek
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

2. ✅ Ejecutar pruebas con el modelo DeepSeek
```bash
python deepseek_test.py
```

3. ✅ Configurar alertas en AWS CloudWatch para monitoreo de uso de GPU y CPU.

---

## **📌 Conclusión**
Con esta configuración, la instancia AWS EC2 está lista para ejecutar **DeepSeek LLM** con **soporte para GPU NVIDIA y Docker**, asegurando un entorno optimizado para pruebas de IA.

🚀 **¡Todo listo para desplegar modelos de Deep Learning en la nube!**

---

## **📌 Referencias y Recursos**
- 📄 [Documentación Oficial de AWS EC2](https://docs.aws.amazon.com/ec2/)
- 📄 [Guía de NVIDIA Docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
- 📄 [Repositorio Oficial de DeepSeek-AI](https://github.com/DeepSeek-AI/DeepSeek-LLM)

---

## **📌 Contacto y Soporte**
Si necesitas asistencia adicional:
- 📧 Contacto: `tuemail@ejemplo.com`
- 🛠 Foro de AWS: [AWS Forums](https://forums.aws.amazon.com/)
- 💬 Comunidad de Deep Learning: [Deep Learning Discord](https://discord.gg/deep-learning)

---

## **📌 Notas Finales**
✅ **Recuerda** cerrar la instancia cuando no la estés utilizando para evitar costos innecesarios.  
✅ **Para futuras mejoras**, considera probar instancias más potentes como `g5.xlarge` si el modelo requiere más VRAM.  
✅ **Explora CloudWatch** para monitorear el rendimiento y establecer alertas en caso de uso excesivo de GPU o CPU.

📌 **Última actualización:** _$(date)_  

---

✍ **Autor:** _Fabio Salinas_  
📅 **Fecha de Creación:** _2025-02-22_  
🏢 **Proyecto:** _Multi LLM_
