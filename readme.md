1. Get Started with EC2
Go to https://aws.amazon.com/ec2/ and click the orange Get started with Amazon EC2 button. Sign up for an AWS account if you don’t already have one.

Otherwise, navigate to the EC2 Dashboard and click on Launch instance. Search for and select Ubuntu Server 20.04. Feel free to select any instance type, but to stay within the Free Tier, select t2.micro.

Continue through the setup keeping the defaults except for the following options:

Add Storage: 8 GB
Configure Security Group: HTTP from Anywhere
Configure Security Group: HTTPS from Anywhere
Click on the launch button, and in the prompt select Create a new key pair called wp-key.pem and download the key pair. It will be a pem file.

Finally click on the blue Launch Instance button.

A few moments later, the Instance state column will say running which means it’s online and you can proceed to the next step.

2. Link Domain Name
If you have a domain name, you can link it to your EC2 instance by creating a DNS record.

Go to your registrar and find the DNS settings of your domain name. Create an A record that points to the IP address of your EC2 instance and another A record for the www version of your website that points to the same IP address of your EC2 instance.

If you’re not familiar with this process, learn more about DNS A records here.

3. Login to EC2 via SSH
If you are on Mac or Linux, you can use Terminal to login via SSH and Windows users can either use git bash login. Here is an example of the SSH command to login to your EC2 server.


ssh -i wp-key.pem ubuntu@IP
If you configured your DNS settings in the previous step, you can also use your domain name instead of your instance IP address in the command above.

4. Update System and Install LEMP Packages
Execute the following to upgrade Ubuntu server packages.

sudo apt update
sudo apt upgrade
Use the apt package manager to install PHP, MariaDB, and the Nginx web server.

sudo apt install nginx mariadb-server php-fpm php-mysql
5. Install WordPress
After logging in to your server as described above, execute the following commands to install WordPress on Ubuntu.

cd /var/www

sudo wget https://wordpress.org/latest.tar.gz

sudo tar -xzvf latest.tar.gz

sudo rm latest.tar.gz

sudo chown -R www-data:www-data wordpress

sudo find wordpress/ -type d -exec chmod 755 {} \;

sudo find wordpress/ -type f -exec chmod 644 {} \;

6. Setup the Database
Secure your MariaDB installation by adding a password and disabling other features. When prompted, answer Y.

sudo mysql_secure_installation
Access the MariaDB console with the password that you just created.

sudo mysql -u root -p

Within the MariaDB console, create a database for WordPress. Please choose your own database name, user name, and a password.

create database wp_db default character set utf8 collate utf8_unicode_ci;

create user 'wp_user'@'localhost' identified by 'admin@123';

grant all privileges on wp_db.* TO 'wp_user'@'localhost';

flush privileges;
exit
7. Configure Nginx Web Server
Navigate to the directory which contains configuration files for the Nginx web server, and create a new configuration file with the text editor of your choice. In this i use the text editor is vim.

cd /etc/nginx/sites-available/

sudo vim wordpress.conf

Use this configuration as a template for your website. Please change the server_name and make sure that the php-handler socket exists (you may have a different version of PHP installed).


upstream php-handler {
        
        server unix:/var/run/php/php8.3-fpm.sock;
}

server {
       
        listen 80;
        
        server_name amirs.myddns.me;
        
        root /var/www/wordpress;
        
        index index.php;

        location / {
              
                try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
               
                include snippets/fastcgi-php.conf;
                
                fastcgi_pass php-handler;
        }
}

Make a symbolic link to tell Nginx about your website, and apply the changes by restarting the web server.

sudo ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/

sudo nginx -t

sudo systemctl restart nginx
