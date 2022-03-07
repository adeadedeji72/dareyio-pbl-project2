## **WEB STACK IMPLEMENTATION (LEMP STACK)** ##

### **Step 0 – Preparing prerequisites** ###

Spine a new EC2 Ubuntu 20.04 instance like was one in Project 1

Connect to the EC2 instance with the ssh-key from AWS

Use any  terminal emulator you are comfortable with

Login to your EC2 instance, you will have an interface like this:

![](lempserver-login.jpg)


### **STEP 1 – INSTALLING THE NGINX WEB SERVER** ###

Run these commands, to update the OS
```
sudo apt update
```
and install nginx
```
sudo apt install nginx -y
```

To verify that nginx was successfully installed and is running as a service in Ubuntu, run:
```
sudo systemctl status nginx
```

Retrieve your Public IP address, rather than check it in AWS Web console with: 
```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
You will see the page below in your browser if everything has gone well

![](nginx-default-page.jpg)


### **STEP 2 — INSTALLING MYSQL** ###

Install MYSQL with the command below
```
sudo apt install mysql-server -y
```

Secure the database with the script below. Provide appropriate responses
```
sudo mysql_secure_installation
```

Test if you are able to login with
```
sudo mysql
```
Exit mysql with
```
mysql> exit
```

### **STEP 3 – INSTALLING PHP** ###

You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

You’ll need to install *php-fpm*, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need *php-mysql*, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.
```
sudo apt install php-fpm php-mysql
```

### **STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR** ###

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use **projectLEMP** as an example domain name.

On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.

Create the root web directory for **projectLEMP** as follows
```
sudo mkdir /var/www/projectLEMP
```

Change the owner to the current user
```
sudo chown -R $USER:$USER /var/www/projectLEMP
```
Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:
```
sudo nano /etc/nginx/sites-available/projectLEMP
```

Copy and paste the code below into the empty file created above
```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
Here is what the code above mean

```
*listen* — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
*root* — Defines the document root where the files served by this website are stored.
*index* — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
*server_name* — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
*location /* — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
*location ~ \.php$* — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
*location ~ /\.ht* — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
```