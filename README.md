**The task:**


>Deploy system that consists of Wordpress and MySQL database using Docker technology.
>
>Wordpress requirements:
>
>•	Version: 4.9.3
>
>•	Web server: nginx
>
>There is no specific requirements to MySQL database.
>
>Create free accounts on https://papertrailapp.com and https://www.datadoghq.com using https://10minutemail.com/ mail service.
Configure Nginx and MySQL to send logs to papertrailapp and app specific metrics to datadoghq.
All code should be committed to github.com or gitlab.com public repo.
>
>Provide:
>
>•	Path to github.com or gitlab.com public repo that contains:
>
>o	Dockerfile if such requires
>
>o	Docker compose file if such requires
>
>o	Nginx and MySQL conf files if such requires
>
>o	Instruction how to deploy Wordpress with MySQL on local machine
>
>•	User/Password to papertrailapp and datadoghq accounts.

**Comments:**

Nginx+Wordpress+Mysql(maridb) docker-based system has been built with help of docker-compose.yml file. Please, get acquanted with details below:
```sh

version: '3.7'

# docker secrets to provide some basic data hide
secrets:
    db_pass:
        file: ./db_pass.txt
    db_user:
        file: ./db_user.txt

services:
    nginx:
        image: 'nginx:latest'
        container_name: nginx  # for my convenience I named the container
        restart: always        # whatever happens, container should be always relaunched
        logging:
            driver: syslog     # the syslog driver the one that works with a docker and transfer data outside
            options:
                syslog-address: "udp://logs6.papertrailapp.com:13439" # path to the Papertrail which coolects the logs
                tag: "nginx"   # for convenience, we'll mark all nginx logs as "nginx" (by default containers ID is used)
        ports:
            - '80:80'
        links:
            - wordpress        # wordpress container, installed and running on nginx server
        volumes:
            - ./nginx/app.conf:/etc/nginx/conf.d/app.conf  # It is important to add php-mudule, so we edit nginx conf file
            - ./public:/var/www/html  # folder that contains web content, has been added for testing purpose

    wordpress:    # image tag that consist of wordpress 4.9.3 version + fpm-server (works with php)
        image: 'wordpress:4.9.3-php7.2-fpm'  
        container_name: wordpress
        restart: always
        logging:
            driver: syslog
            options:
                syslog-address: "udp://logs6.papertrailapp.com:13439"
                tag: "wordpress"
        links:
            - mysql # wordpress connected to mysql database
        ports:
            - 8080:80  # in order to avoid confilct default port 80 has been replaced by 8080
        environment:
            - WORDPRESS_DB_HOST=mysql  # db container name
            - WORDPRESS_DB_USER_FILE=/run/secrets/db_user  # docker secrets hidden data, 
            - WORDPRESS_DB_PASSWORD_FILE=/run/secrets/db_pass  # dockers secrets hidden data, 
            - WORDPRESS_DB_NAME=wordpress
        volumes:
            - ./public:/var/www/html
        secrets:          # dockers secrets has been applied
            - db_pass
            - db_user

    mysql:
        image: 'mariadb'
        restart: always
        container_name: mysql
        logging:
             driver: syslog
             options:
                 syslog-address: "udp://logs6.papertrailapp.com:13439"
                 tag: "mysql"
        ports:
            - '3306:3306'
        volumes:
            - ./db-data:/var/lib/mysql   # volume has been created for testing purpose, not a mandatory thing
            - ./logs:/var/log            # volume has been created for testing purpose, not a mandatory thing
        environment:
            - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_pass
        secrets:
            - db_pass

# container below is a mandatory container to collect data for DataDogHQ metrics

    dd-agent:
        image: 'store/datadog/agent:6.4.1'
        container_name: dd-agent
        restart: always
        links:
            - nginx
            - mysql
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock:ro
            - /proc/:/host/proc/:ro
            - /sys/fs/cgroup/:/host/sys/fs/cgroup:ro
        environment:
            - DD_API_KEY=9222ef752bb7c5820a9a57a44e6aaaad # key which allows data to be sent to appropriate (mine) account
```

Here below you can find nginx configuration file which main purpose is to enable php module and set index.php as the default page.

```sh
server {

	listen 80;   
	server_name _;

        index index.php index.html; # so here index.php stands in front of index.html which means it will be requested before the index.html in case if its found, once we get "/" requesst.
	root /var/www/html; # container webcontent folder 
 
        location / {
                try_files $uri $uri/ /index.php?$args =404;
        }
	
	# the below location directive connects php-module when php requests arrive
        location ~ \.php$ {
                include fastcgi_params;
                fastcgi_index index.php; # index.php - defauly page to open
                fastcgi_pass  wordpress:9000; # wordpress container which is actually Wordpress+FPM (fpm web-server that's working with php requests)
                fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name; (path to the scrip file which is in our case named "index.php")
	}
}
```


This part of the task here above is represented by ***docker-compose.yml*** file and ***nginx*** folder with conf file inside.

***cred.zip*** file is password protected and contains papertail and datadoghq credentials as well as docker secret files.
