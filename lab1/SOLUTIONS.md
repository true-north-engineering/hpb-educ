# Lab 1 - Containers

Be aware that these solutions are for ***user50***. Please adjust all paths for your username!

## Task 1 - Create mysql container

1. Go to public DockerHub repo and find mysql 5.5 container image. Go through the documentation and find which environment variables are used for database configuration, especially MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE.

2. Create podman network mynet using bridge driver.

```podman network create -d bridge mynet```

3. Create a directory for mysql data in /home/<your_username>/mysql

```mkdir /home/user50/mysql```

4. Create container which has the following properties:
    * Container name is mysql
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE to values of your choosing
    * Container is attached to mynet network
    * Container mounts /home/<your_username>/mysql on host system to /var/lib/mysql folder in the container
    * Container image used is mysql:5.5

```podman run -d --name mysql -e MYSQL_ROOT_PASSWORD=r00tpassw0rd -e MYSQL_USER=todo -e MYSQL_PASSWORD=todopassw0rd -e MYSQL_DATABASE=todo --network mynet -v /home/user50/mysql:/var/lib/mysql:Z mysql:5.5```

5. List the running containers

```podman ps```

6. Observe the logs of mysql container

```podman logs mysql```

7. Exec into the container, connect to the database specified with environment variables and create the ***Item*** table
```
podman exec -it mysql bash

# Command for connecting to the database
mysql -h localhost -u<mysql_username> -p<mysql_password> <mysql_database>

# Create the table
create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));

# Check if table was created
show tables;

# Check if select works
select * from Items;
```

## Task 2 - Write a Containerfile for nodejs application and run the application

1. The Containerfile should satisfy the following requirements
    * The location of Containerfile should be ~/hpb-educ/lab1/nodejs/Containerfile
    * Base image should be node:5
    * WORKDIR should be /home/node/app
    * Copy the contents of nodejs folder into WORKDIR
    * Run ```npm install```
    * Set CMD to run ```node app.js```

```
cd ~/hpb-educ/lab1/nodejs
vi Containerfile
```

Content of the Containerfile:

```
FROM node:5

WORKDIR /home/node/app
COPY . /home/node/app
RUN npm install

CMD ["node", "app.js"]
```

2. Build the Containerfile and name the image todo:latest

```podman build -t todo:latest .```

3. List and inspect the builded image

```
podman images
podman inspect todo:latest
```

4. Create container with builded image. The container should have the following properties:
    * Container name is todo
    * Container is running in background (detached mode)
    * Set the values od the following environment variables MYSQL_ENV_MYSQL_DATABASE, MYSQL_ENV_MYSQL_USER, MYSQL_ENV_MYSQL_PASSWORD to values you have specified when starting mysql container.
    * Container is attached to mynet network
    * Container image used is todo:latest
    * Publish port 30080

```podman run -d -p 30080:30080 --network foo -e MYSQL_ENV_MYSQL_DATABASE=todo -e MYSQL_ENV_MYSQL_USER=todo -e MYSQL_ENV_MYSQL_PASSWORD=todopassw0rd --name todo todo:latest```