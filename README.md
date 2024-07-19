# FinalExamProof

ctrl + D Salir de root

1-Creo una instancia EC2 con clave de accesos y un repo en git

ssh-keygen -t ed25519 -C "your_email@example.com"

2-Instalaciones en EC2

# GIT

sudo yum update

sudo yum install git

git -v

# DOCKER

sudo yum install -y docker 

Inicie el servicio Docker. 

sudo service docker start 

Agregue el ec2-user al grupo docker para que pueda ejecutar comandos de Docker sin usar sudo. 

sudo usermod -a -G docker ec2-user 

# DOCKER COMPOSE

sudo curl -L "https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# TERRAFORM

sudo yum install -y yum-utils

sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo

sudo yum -y install terraform

3- Construir y Levantar los Contenedores

Ejecuta los siguientes comandos desde el directorio FinalExamProof:

docker-compose build

docker-compose up -d
