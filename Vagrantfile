Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  # Configurar red en modo puente
  config.vm.network "public_network", bridge: "Realtek PCIe GbE Family Controller"

  # Configuración del proveedor para VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096" # 4 GB de RAM
    vb.cpus = 5        # 5 CPUs
  end

  # Script de aprovisionamiento
  config.vm.provision "shell", inline: <<-SHELL
    set -e

    # Actualizar el sistema
    echo "Actualizando el sistema..."
    apt update -y && apt upgrade -y

    # Instalar Docker
    echo "Instalando Docker..."
    curl -fsSL https://get.docker.com -o get-docker.sh
    bash get-docker.sh

    # Instalar Docker Compose
    echo "Instalando Docker Compose..."
    mkdir -p /usr/local/lib/docker/cli-plugins/
    curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 \
      -o /usr/local/lib/docker/cli-plugins/docker-compose
    chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

    # Validar las versiones instaladas
    docker --version || { echo "Error: Docker no instalado correctamente."; exit 1; }
    docker compose version || { echo "Error: Docker Compose no instalado correctamente."; exit 1; }

    # Crear la carpeta DockerFiles
    echo "Creando carpeta DockerFiles..."
    mkdir -p /DockerFiles && chmod 755 /DockerFiles

    # Validar y copiar archivos
    echo "Copiando archivos a DockerFiles..."
    if [ -f /vagrant/dockerfile ] && [ -f /vagrant/docker-compose.yaml ]; then
        cp /vagrant/dockerfile /DockerFiles/
        cp /vagrant/docker-compose.yaml /DockerFiles/
    else
        echo "Error: Los archivos dockerfile o docker-compose.yaml no existen en /vagrant."
        exit 1
    fi

    # Cambiar de directorio a DockerFiles
    cd /DockerFiles

    # Construir la imagen de Docker
    echo "Construyendo imagen de Docker..."
    docker build -t myjenkins-blueocean:2.479.1-1 .

    # Despliegue de contenedores usando Docker Compose
    echo "Desplegando contenedores con Docker Compose..."
    docker compose up -d

    # Esperar a que Jenkins se inicie
    echo "Esperando a que Jenkins se inicialice..."
    sleep 30

    # Obtener IP pública
    PUBLIC_IP=$(hostname -I | awk '{print $2}')

    # Obtener contraseña inicial de Jenkins
    echo "Obteniendo contraseña inicial de Jenkins..."
    JENKINS_PASSWORD=$(docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword || echo "No se pudo obtener la contraseña de Jenkins.")

    # Validar estado de Zabbix
    echo "Validando estado de Zabbix..."
    ZABBIX_STATUS=$(docker ps | grep zabbix-web-nginx-pgsql | awk '{print $NF}')
    if [ "$ZABBIX_STATUS" != "" ]; then
        ZABBIX_PORT="8084"
    else
        ZABBIX_PORT="No disponible (Zabbix no está corriendo)"
    fi

    # Mostrar información final
    echo "========================================="
    echo "Despliegue completado con éxito."
    echo "IP Pública del Servidor: $PUBLIC_IP"
    echo "Acceso a Jenkins: http://$PUBLIC_IP:8080"
    echo "Contraseña Inicial de Jenkins: $JENKINS_PASSWORD"
    echo "Acceso a Zabbix: http://$PUBLIC_IP:$ZABBIX_PORT"
    echo "========================================="
  SHELL
end
