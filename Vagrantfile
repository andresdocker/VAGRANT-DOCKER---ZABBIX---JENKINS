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
    # Actualizar el sistema
    apt update -y && apt upgrade -y

    # Instalar Docker
    curl -fsSL https://get.docker.com -o get-docker.sh
    bash get-docker.sh

    # Instalar Docker Compose
    mkdir -p /usr/local/lib/docker/cli-plugins/
    curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 \
      -o /usr/local/lib/docker/cli-plugins/docker-compose
    chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

    # Validar las versiones instaladas
    docker --version || { echo "Error: Docker no instalado correctamente."; exit 1; }
    docker compose version || { echo "Error: Docker Compose no instalado correctamente."; exit 1; }

    # Crear la carpeta DockerFiles
    mkdir -p /DockerFiles && chmod 755 /DockerFiles

    # Validar y copiar archivos
    if [ -f /vagrant/dockerfile ] && [ -f /vagrant/docker-compose.yaml ]; then
        cp /vagrant/dockerfile /DockerFiles/
        cp /vagrant/docker-compose.yaml /DockerFiles/
    else
        echo "Error: Los archivos dockerfile o docker-compose.yaml no existen en /vagrant."
        exit 1
    fi

    # Validar la existencia de la carpeta y cambiar de directorio
    if [ -d /DockerFiles ]; then
        cd /DockerFiles
    else
        echo "Error: La carpeta /DockerFiles no existe."
        exit 1
    fi

    # Construir la imagen de Docker
    if [ -f dockerfile ]; then
        docker build -t myjenkins-blueocean:2.479.1-1 .
    else
        echo "Error: dockerfile no encontrado en /DockerFiles."
        exit 1
    fi

    # Despliegue de contenedores usando Docker Compose
    if [ -f docker-compose.yaml ]; then
        docker compose up -d
    else
        echo "Error: docker-compose.yaml no encontrado en /DockerFiles."
        exit 1
    fi

    echo "Despliegue completado con éxito."
  SHELL
end
