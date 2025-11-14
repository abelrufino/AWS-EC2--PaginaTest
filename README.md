# AWS-EC2--PaginaTest
Iniciar sua instância do EC2 

Esse script faz o deploy automático do seu site quando a instância EC2 é iniciada. Ele instala o Apache, clona o repositório e exibe a página diretamente no navegador via IP público da instância.

#!/bin/bash
# Update packages
sudo yum update -y

# Install Git and Apache
sudo yum install -y git httpd

# Start and enable Apache
sudo systemctl start httpd
sudo systemctl enable httpd

# Clone the repository
cd /var/www/html
sudo git clone https://github.com/abelrufino/AWS-EC2--PaginaTest.git
sudo cp -r AWS-EC2--PaginaTest/* /var/www/html/

# ===== Get IMDSv2 token =====
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600" -s)

# ===== Fetch metadata =====
AZ=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
-s http://169.254.169.254/latest/meta-data/placement/availability-zone)
REGION=${AZ::-1}

# ===== Insert into the page =====
sudo sed -i "s/<span id=\"region\" class=\"font-semibold text-cyan-300\">Detecting...<\/span>/<span id=\"region\" class=\"font-semibold text-cyan-300\">${REGION}<\/span>/g" /var/www/html/index.html

sudo sed -i "s/<span id=\"az\" class=\"font-semibold text-cyan-300\">Detecting...<\/span>/<span id=\"az\" class=\"font-semibold text-cyan-300\">${AZ}<\/span>/g" /var/www/html/index.html

# Permissions
sudo chown -R apache:apache /var/www/html
sudo systemctl restart httpd
