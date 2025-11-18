📘 README — Deploying the CodeKloud E-Commerce App on Ubuntu (WSL)

This document provides step-by-step instructions to deploy the CodeKloud E-Commerce Application on Ubuntu (WSL or regular Ubuntu Server).
This version is adapted from the original CentOS guide.

🟩 1. Update System
sudo apt update -y
sudo apt upgrade -y

🟩 2. Install & Configure MariaDB (Database Layer)
Install MariaDB
sudo apt install -y mariadb-server

Start & Enable MariaDB

(If WSL supports systemd)

sudo systemctl start mariadb
sudo systemctl enable mariadb

Login to MySQL
sudo mysql

Create database & user

Inside MySQL shell:

CREATE DATABASE ecomdb;
CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
FLUSH PRIVILEGES;

🟩 3. Load Initial Product Data

Create the SQL file:

cat > db-load-script.sql << 'EOF'
USE ecomdb;
CREATE TABLE products (
    id mediumint(8) unsigned NOT NULL auto_increment,
    Name varchar(255) default NULL,
    Price varchar(255) default NULL,
    ImageUrl varchar(255) default NULL,
    PRIMARY KEY (id)
) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES
("Laptop","100","c-1.png"),
("Drone","200","c-2.png"),
("VR","300","c-3.png"),
("Tablet","50","c-5.png"),
("Watch","90","c-6.png"),
("Phone Covers","20","c-7.png"),
("Phone","80","c-8.png"),
("Laptop","150","c-4.png");
EOF


Load data:

sudo mysql < db-load-script.sql

🟩 4. Install Apache & PHP (Web Layer)

Install required packages:

sudo apt install -y apache2 php php-mysql git


Make sure Apache loads index.php first:

sudo sed -i 's/index.html/index.php/g' /etc/apache2/mods-enabled/dir.conf


Restart Apache:

sudo systemctl restart apache2

🟩 5. Deploy Application Code

Clone the repository into Apache’s web directory:

sudo rm -rf /var/www/html/*
sudo git clone https://gitlab.com/debjyotimt/e-commerce-app-3-tier.git /var/www/html

🟩 6. Create Environment File

Create a .env in /var/www/html:

sudo bash -c 'cat > /var/www/html/.env << "EOF"
DB_HOST=localhost
DB_USER=ecomuser
DB_PASSWORD=ecompassword
DB_NAME=ecomdb
EOF'

🟩 7. Application Code Notes

The provided index.php already contains code to read values from the environment variables:

$dbHost = getenv('DB_HOST');
$dbUser = getenv('DB_USER');
$dbPassword = getenv('DB_PASSWORD');
$dbName = getenv('DB_NAME');


No additional changes are required.

🟩 8. Test the Application

From WSL terminal:

curl http://localhost


From Windows browser:

http://localhost


You should now see the product list loading from MariaDB.