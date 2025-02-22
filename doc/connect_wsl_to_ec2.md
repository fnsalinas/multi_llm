# Conectar WSL2 (Ubuntu) a una máquina EC2 para trabajar de forma remota

Este documento proporciona una guía paso a paso para conectar tu entorno WSL2 (Ubuntu) a una instancia de EC2 y trabajar de forma remota utilizando Visual Studio Code.

## Prerrequisitos

1. Tener una instancia de EC2 en AWS configurada y en ejecución.
2. Tener WSL2 instalado en tu máquina con Ubuntu.
3. Tener Visual Studio Code instalado en tu máquina.
4. Tener la extensión de Remote - WSL y Remote - SSH instaladas en Visual Studio Code.
5. Tener una clave privada (.pem) para acceder a tu instancia de EC2.

## Pasos para la conexión

### 1. Configurar la instancia de EC2

1. Inicia sesión en tu cuenta de AWS.
2. Navega a la consola de EC2 y selecciona tu instancia.
3. Asegúrate de que tu instancia esté en ejecución.
4. Anota la dirección IP pública de tu instancia.

### 2. Configurar WSL2

1. Abre tu terminal de WSL2 (Ubuntu).
2. Asegúrate de que tu sistema esté actualizado:
    ```sh
    sudo apt update && sudo apt upgrade
    ```

### 3. Conectar a la instancia de EC2 desde WSL2

1. Navega al directorio donde se encuentra tu clave privada (.pem):
    ```sh
    cd /path/to/your/keypair
    ```
2. Cambia los permisos de la clave privada para que solo el propietario pueda leerla:
    ```sh
    chmod 400 your-key-pair.pem
    ```
3. Conéctate a tu instancia de EC2 utilizando SSH:
    ```sh
    ssh -i your-key-pair.pem ubuntu@your-ec2-ip
    ```
    Reemplaza `your-key-pair.pem` con el nombre de tu archivo de clave privada y `your-ec2-ip` con la dirección IP pública de tu instancia de EC2.

### 4. Configurar Visual Studio Code para trabajar de forma remota

1. Abre Visual Studio Code.
2. Presiona `Ctrl+Shift+P` para abrir la paleta de comandos.
3. Escribe `Remote-SSH: Connect to Host...` y selecciona la opción.
4. Ingresa la dirección de tu instancia de EC2 en el formato `ubuntu@your-ec2-ip` y presiona Enter.
5. Selecciona el archivo de clave privada (.pem) cuando se te solicite.
6. Visual Studio Code se conectará a tu instancia de EC2 y podrás trabajar de forma remota.

### 5. Configurar el entorno de desarrollo en la instancia de EC2

1. Una vez conectado a la instancia de EC2 desde Visual Studio Code, abre una terminal integrada.
2. Instala las dependencias necesarias para tu proyecto.
3. Configura tu entorno de desarrollo según las necesidades de tu proyecto.

## Conclusión

Siguiendo estos pasos, podrás conectar tu entorno WSL2 (Ubuntu) a una instancia de EC2 y trabajar de forma remota utilizando Visual Studio Code. Esto te permitirá aprovechar la potencia de la nube mientras trabajas en un entorno familiar.