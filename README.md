# Hosting Spring boot + Swagger project
## Software requirement 
  - Nginx ( reverse proxy )
  - Mysql or Other database base on project
  - openjkd-21-jdk ( base on project requirement )
## Setup 
### install mysql-server and Create DB for Spring boot
  ```
  sudo apt update
  sudo apt install mysql-server -y
  sudo mysql_secure_installation
  ```
  ```
  Create Database
  ```
### Configure Jar file run as service in linux
  - install java on linux server ( sudo apt install openjdk-xx-jdk ) xx = java version Ex: openjdk-21-jdk
  - upload jar file to linux server
  - Create Service File to run jar file ( sudo nano /etc/systemd/system/spring_boot.service ) 
  - if spring boot has image upload to /opt/myApp/static
    ```
      [Unit]
        Description =Java Spring Boot App
        After=network.target
      [Service]
        User=ubuntu #( can change to user your wnat to run spring boot with )
        WorkingDirectory=/home/ubuntu/java_api #( replace your jar directory )
        PermissionsStartOnly=true
        ExecStartPre=/bin/mkdir -p /opt/myApp/static
        ExecStartPre=/bin/chown -R username:username /opt/myApp #( replace username with your user )
        ExecStartPre=/bin/chmod -R 755 /opt/myApp
        ExecStart=java -jar api.jar
        Restart = always
      [Install]
        WantedBy=multi-user.target
    ```
  - if spring boot has image upload the same folder jar file
    ```
      [Unit]
        Description =Java Spring Boot App
        After=network.target
      [Service]
        User=ubuntu #( can change to user your wnat to run spring boot with )
        WorkingDirectory=/home/ubuntu/java_api #( replace your jar directory )
        ExecStartPre=/bin/mkdir -p /home/ubuntu/java_api/myApp/static #( replace your jar directory )
        ExecStartPre=/bin/chmod -R 775 /home/ubuntu
        ExecStart=java -jar api.jar
        Restart = always
      [Install]
        WantedBy=multi-user.target
    ```
  - Relaod systemd and start spring boot service
      ```
      sudo systemctl daemon-reload
      sudo systemctl enable spring_boot.service
      sudo systemctl start spring_boot.service
      sudo systemctl status spring_boot.service
      ```
   - test access spring boot + swagger via http://serverip:8080/swagger-ui/index.html
### Install nginx and configure
   ```
  sudo apt update
  sudo apt install nginx -y
  ```
- create file for nginx hosting ( sudo nano /etc/nginx/site-available/spring_boot.conf )
  - if spring boot has image upload to /opt/myApp/static
    ```
        server {
            listen 443 ssl;
            server_name cors.setecist.uk; #( replace your own domain name or Server ip )
        
            ssl_certificate /etc/ssl/cf/certificate.pem; #( replace your ssl certificate path )
            ssl_certificate_key /etc/ssl/cf/certificatekey.pem; #( replace your ssl certificate key path )
            location ^~ /static/ {
                alias /opt/myApp/static;
                autoindex off;
                expires 30d;
                add_header Cache-Control "public";
                add_header Access-Control-Allow-Origin *;
                try_files $uri =404;
            }
        
            location = / {
                return 301 /swagger-ui/index.html;
            }
        
            location / {
                proxy_pass http://127.0.0.1:8080;
        
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
                proxy_set_header X-Forwarded-Port 443;
            }
        }
    ```
  - if spring boot has image upload the same folder jar file
    ```
             server {
            listen 443 ssl;
            server_name cors.setecist.uk;  #( replace your own domain name or Server ip )
        
            ssl_certificate /etc/ssl/cf/certificate.pem; #( replace your ssl certificate path )
            ssl_certificate_key /etc/ssl/cf/certificatekey.pem; #( replace your ssl certificate key path )
            location ^~ /static/ {
                alias /home/ubuntu/java_api/myApp/static/; #( replace your current directory )
                autoindex off;
                expires 30d;
                add_header Cache-Control "public";
                add_header Access-Control-Allow-Origin *;
                try_files $uri =404;
            }
        
            location = / {
                return 301 /swagger-ui/index.html;
            }
        
            location / {
                proxy_pass http://127.0.0.1:8080;
        
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
                proxy_set_header X-Forwarded-Port 443;
            }
        }
    ```
    
