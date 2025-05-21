# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # 1. Box Configuration
  # --------------------
  config.vm.box = "bento/ubuntu-22.04"
  config.vm.hostname = "minikube-dev-vm"

  # 2. Provider Configuration (VirtualBox es el default)
  # ----------------------------------------------------
  # No configuraremos explícitamente RAM ni CPUs, dejando los defaults del provider.
  # VirtualBox por defecto suele asignar 1 CPU y 512MB o 1024MB de RAM.
  # ¡IMPORTANTE! Minikube necesita más recursos que eso.
  # Es probable que necesites ajustar los recursos de Minikube al iniciarlo
  # con 'minikube start --cpus 2 --memory 2048m' (o similar) DENTRO de la VM.
  # Si la VM es demasiado lenta o Minikube no arranca bien, considera descomentar
  # y ajustar las líneas de memoria/cpu para la VM de Vagrant si tu sistema lo permite.
  # config.vm.provider "virtualbox" do |vb|
  #   vb.memory = "3072"  # Ejemplo: 3GB para la VM, dejando algo para tu host
  #   vb.cpus = "2"
  # end

  # 3. Network Configuration (Opcional)
  # ---------------------------------------------
  config.vm.network "private_network", ip: "192.168.56.10"

  # 4. Shared Folders
  # -----------------
  # Vagrant sincroniza automáticamente la carpeta que contiene este Vagrantfile
  # con /vagrant en la máquina virtual invitada. No necesitas añadir nada
  # extra para la carpeta por defecto.
  # Si quisieras añadir otra carpeta específica:
  config.vm.synced_folder "./", "/vagrant"

  # 5. Provisioning
  # ---------------
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    echo ">>> Actualizando lista de paquetes..."
    apt-get update -y

    echo ">>> Instalando prerrequisitos para Docker y otras herramientas..."
    apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release \
        software-properties-common \
        conntrack # A menudo necesario para Minikube

    echo ">>> Instalando Docker..."
    install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    chmod a+r /etc/apt/keyrings/docker.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    apt-get update -y
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    echo ">>> Añadiendo usuario 'vagrant' al grupo 'docker'..."
    usermod -aG docker vagrant

    echo ">>> Instalando kubectl..."
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    rm kubectl

    echo ">>> Instalando Minikube..."
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    install minikube /usr/local/bin/
    rm minikube

    echo ">>> Configurando bash completion para kubectl y minikube (para el usuario vagrant)..."
    apt-get install -y bash-completion
    sudo -u vagrant bash -c 'echo "source <(kubectl completion bash)" >> ~/.bashrc'
    sudo -u vagrant bash -c 'echo "source <(minikube completion bash)" >> ~/.bashrc'

    echo "---------------------------------------------------------------------"
    echo "VM aprovisionada y lista para Minikube."
    echo "Tu carpeta actual en el host está compartida en /vagrant dentro de la VM."
    echo "IMPORTANTE: La VM Vagrant tiene recursos limitados por defecto."
    echo "Al iniciar Minikube, considera especificar CPUs y memoria, por ejemplo:"
    echo "   minikube start --driver=docker --cpus=2 --memory=2048m"
    echo "Ajusta estos valores según la capacidad de tu sistema y lo que Minikube necesita."
    echo ""
    echo "Para usarla:"
    echo "1. Sal de esta sesión de aprovisionamiento si aún está activa."
    echo "2. Conéctate a la VM con: vagrant ssh"
    echo "3. Una vez dentro, navega a la carpeta compartida: cd /vagrant"
    echo "4. Inicia Minikube (ej. minikube start --driver=docker --cpus=2 --memory=2048m)"
    echo "5. Para que la bash completion tome efecto, puede que necesites ejecutar:"
    echo "   source ~/.bashrc  O salir y re-entrar a la sesión SSH."
    echo "---------------------------------------------------------------------"
  SHELL

  # 6. SSH User
  # -----------
  config.ssh.username = "vagrant"

end