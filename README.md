# Hosting Spring boot project
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
  - upload jar file to linux server
  - Create Service File to run jar file
  - if spring boot has image upload to /opt/myApp/static
    ```
      [Unit]
        Description =Java Spring Boot App
        After=network.target
      [Service]
        User=ubuntu
        WorkingDirectory=/home/ubuntu/java_api ( replace your jar folder )
        PermissionsStartOnly=true
        ExecStartPre=/bin/mkdir -p /opt/myApp/static
        ExecStartPre=/bin/chown -R username:username /opt/myApp ( replace username with your user )
        ExecStartPre=/bin/chmod -R 755 /opt/myApp
        ExecStart=java -jar api.jar
        Restart = always
      [Install]
        WantedBy=multi-user.target
  

