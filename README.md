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

# MySQL 

sudo yum install https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm
sudo yum install mysql-community-server
sudo systemctl enable --now mysqld
sudo mysql_secure_installation
mysql
mysql -h mysql
docker ps
mysql -h localhost:3306
mysql -h 0.0.0.0:3306
mysql -h 0.0.0.0:3306 -u mysql
docker exec -it 86411fb0de66 -- bash
docker exec -it 86411fb0de66 -- /bin/bash
docker exec -it 86411fb0de66 -- /bin/sh
mysql -h localhost -P 3306 --protocol=tcp -u root
mysql -h localhost -P 3306 --protocol=tcp -u admin
ls
cat docker-compose.yml 
mysql -h localhost -P 3306 --protocol=tcp -u user
mysql -h localhost -P 3306 --protocol=tcp -u user -p
mysql -h localhost -P 3306 --protocol=tcp -u root -p

----------------------------------------------------------------------------------------

Para adaptar el proyecto de Docker Compose a Terraform y desplegarlo en AWS utilizando los recursos gestionados por Terraform, debemos seguir estos pasos:

# Configurar AWS CLI y Terraform

Definir la infraestructura en Terraform

Configurar Terraform para crear una instancia EC2 y un RDS MySQL

Configurar Docker en la instancia EC2

Desplegar la aplicación PHP en la instancia EC2

Configurar las variables de entorno y la conexión a la base de datos

**Paso 1: Configurar AWS CLI y Terraform**

Instalar AWS CLI:

pip install awscli
aws configure

**Instalar Terraform:**

sudo yum install -y yum-utils

sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo

sudo yum -y install terraform

terraform --version

**Construir y Levantar los Contenedores**

Ejecuta los siguientes comandos desde el directorio FinalExamProof:

docker-compose build

docker-compose up -d

**Definir la infraestructura en Terraform**

Crear el archivo main.tf:


provider "aws" {
  region = "us-west-2"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "main" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-west-2a"
}

resource "aws_security_group" "web" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "db" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_db_instance" "default" {
  allocated_storage    = 20
  storage_type         = "gp2"
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t2.micro"
  name                 = "mydatabase"
  username             = "admin"
  password             = "admin"
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot  = true
  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name = aws_db_subnet_group.main.name
}

resource "aws_db_subnet_group" "main" {
  name       = "main"
  subnet_ids = [aws_subnet.main.id]

  tags = {
    Name = "Main subnet group"
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main.id
  security_groups = [aws_security_group.web.name]

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              amazon-linux-extras install docker -y
              service docker start
              usermod -a -G docker ec2-user
              docker run -d -p 80:80 myproject/apache-php
              EOF

  tags = {
    Name = "WebServer"
  }
}

output "web_instance_public_ip" {
  value = aws_instance.web.public_ip
}

**Paso 3: Configurar Terraform para crear una instancia EC2 y un RDS MySQL**

Inicializar Terraform:

terraform init

Crear un plan de ejecución:

terraform plan

Aplicar el plan:

terraform apply

**Paso 4: Configurar Docker en la instancia EC2**

Terraform ejecutará el script user_data durante la creación de la instancia EC2, lo que instalará Docker y desplegará el contenedor de tu aplicación PHP.

**Paso 5: Desplegar la aplicación PHP en la instancia EC2**

Crear un repositorio de Docker Hub o ECR y subir tus imágenes de Docker:

docker login

docker build -t myproject/apache-php ./PROJECT/apache-php

docker tag myproject/apache-php:latest <your-dockerhub-username>/myproject/apache-php:latest

docker push <your-dockerhub-username>/myproject/apache-php:latest

Modificar el script user_data en main.tf para usar la imagen desde Docker Hub o ECR:

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              amazon-linux-extras install docker -y
              service docker start
              usermod -a -G docker ec2-user
              docker run -d -p 80:80 <your-dockerhub-username>/myproject/apache-php:latest
              EOF

**Paso 6: Configurar las variables de entorno y la conexión a la base de datos

Actualizar tu archivo index.php para usar la base de datos RDS:

index.php

$servername = "YOUR_RDS_ENDPOINT";
$username = "admin";
$password = "admin";
$dbname = "mydatabase";

Recrear la infraestructura para aplicar los cambios:

terraform apply

Con estos pasos, logramos que la aplicación PHP despliegue en una instancia EC2 en AWS y conecte a una base de datos MySQL gestionada por RDS, utilizando Terraform para gestionar toda la infraestructura.
