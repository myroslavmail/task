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

**Completed task comments:**

Nginx+Wordpress+Mysql(maridb) docker-based system has been built via docker-compose.yml file. Please, get intruduced with the code and comments below:
```sh



nginx:
    image: 'nginx:latest'
    container_name: myng # for my convenience I named the container "myng"
    restart: always      # whatever happens, container should always be relaunched automatically
    log_driver: syslog   # the syslog driver the one that works with a docker technology and transfer data to syslog server
		log_opt:
      syslog-address: "udp://logs6.papertrailapp.com:13439"  # the url to the Papertrail server which coolects the logs
      tag: "nginx"  # for the convenience container will be recognized as "nginx" (instead of default container ID option)
    ports:
        - '80:80'  
    links:
        - wordpress  # wordpress will be running on nginx web-server
    volumes:
        - ./nginx/app.conf:/etc/nginx/conf.d/app.conf # It is important to adjust .conf file to add php-mudule
        - ./public:/var/www/html # folder that contains all the data, was add for testing purpose in most

wordpress:
    image: 'wordpress:4.9.3-php7.2-fpm' # image tag that consist of wordpress 4.9.3 version + fpm-server (works with php)
    container_name: mywp
    restart: always
    log_driver: syslog 
    log_opt:
      syslog-address: "udp://logsX.papertrailapp.com:XXXXX" # X - hidden digits
      tag: "wordpress" 
    links:
      - mysql # wordpress should be connected to mysql database
    ports:
      - 8080:80 # to avoid confilct with nginx, port has been adjusted to 8080
    environment: 
      - WORDPRESS_DB_HOST=mysql # "mysql" container record for mariadb which working as db host
      - WORDPRESS_DB_USER=# please, type in the user
      - WORDPRESS_DB_PASSWORD=# please, type in the password
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - ./public:/var/www/html 

mysql:
    image: 'mariadb' 
    container_name: myma
    restart: always
    log_driver: syslog
    log_opt:
      syslog-address: "udp://logsX.papertrailapp.com:XXXXX"
      tag: "mysql"
    ports:
      - '3306:3306'
    volumes:
      - ./db-data:/var/lib/mysql # host directory was created for test purpose, not a mndatory operation
      - ./logs:/var/log # host directory was created for test purpose, not a mndatory operation
    environment:
      - MYSQL_ROOT_PASSWORD=#please, type the password

# container below is a mandatory container to collect data for DataDogHQ metrics
dd-agent:
    image: 'store/datadog/agent:6.4.1'
    container_name: mydd
    restart: always
    links:
     - nginx   # was added to make sure data from nginx is collected, not a mandatory option
     - mysql   # was added to make sure data from mysql is collected, not a mandatory option
    volumes: # next 3 rows were are added as per insturction on datadoghq documantation. Briefly it allows dd-agent to collect running containers metrics
     - /var/run/docker.sock:/var/run/docker.sock:ro
     - /proc/:/host/proc/:ro
     - /sys/fs/cgroup/:/host/sys/fs/cgroup:ro
    environment:
     - DD_API_KEY=#please, type the DD API KEY here, provided by DataDotHQ service to connect metrics to appropriate DataDogHQ account
```

Here below you can find nginx configuration file which main purpose is to enable php module and set index.php as the default page.

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
                fastcgi_pass  mywp:9000; # mywp container which is actually Wordpress+FPM (fpm web-server that's working with php requests)
                fastcgi_param SCRIPT_FILENAME /var/www/html/$fastcgi_script_name; (path to the scrip file which is in our case named "index.php")
	}
}



This part of the task here above is represented by ***docker-compose.yml*** file and ***nginx*** folder with conf file inside.

***cred.zip*** file is password protected and contains papertail and datadoghq account credentials.
