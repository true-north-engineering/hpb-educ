# Lab 1 - Containers

Be aware that these solutions are for ***user50***. Please adjust all paths for your username!

## Task 1 - Create mysql container

1. Go to public DockerHub repo and find mysql 5.5 container image. Go through the documentation and find which environment variables are used for database configuration, especially MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE.

2. Create internal podman network mynet using bridge driver.

```podman network create -d bridge --internal mynet```

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
mysql -h localhost -utodo -ptodopassw0rd <mysql_database>

# Create the table
create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));

# Check if table was created
show tables;

# Check if select works
select * from Item;
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

```podman run -d -p 30080 --network mynet -e MYSQL_ENV_MYSQL_DATABASE=todo -e MYSQL_ENV_MYSQL_USER=todo -e MYSQL_ENV_MYSQL_PASSWORD=todopassw0rd --name todo todo:latest```

5. Find out the port on which the application is listening.

```podman ps```

Note the PORTS column of todo container. It has a value like ```0.0.0.0:40589->30080/tcp```. This means that the host port 40589 is redirected to the container's port 30080. In this case the app is awailable outside on port 40589.

6. Test the app by openhing the URL ```http://hpb1.tn.hr:<port>/todo/```. Please note that the last trailing slash (/) is important, so don't omit him.

Open URL http://hpb1.tn.hr:40589/todo/

## Task 3 - Push the image to Nexus

1. Go to https://nexus.ocp.hpb.tn.hr and login with username admin. Password is the same as password for your ssh user on hpb1.tn.hr

2. Login with podman to nexus-docker.ocp.hpb.tn.hr.

```podman login nexus-docker.ocp.hpb.tn.hr```

3. Tag the todo:latest image with podman to nexus-docker.ocp.hpb.tn.hr/todo:user50

```podman tag todo:latest nexus-docker.ocp.hpb.tn.hr/todo:user50```

4. Push the image to Nexus.

```podman push nexus-docker.ocp.hpb.tn.hr/todo:user50```

5. Browse the Nexus through web interface and find your image.

## Task 4 - Cleanup

1. Cleanup everything by stopping the containers, removing them, removing the images and network mynet.

```
podman ps
podman stop todo
podman stop mysql
podman ps
podman ps -a
podman rm -a

podman images
podman rmi -a
podman images -a

podman network ls
podman network rm mynet
podman network ls
```

## Task 5 (Optional) - Build todo app using buildah

1. Build the same todo application from Task 2, Step 1 using Buildah and commit the builded image as todo:latest.

```
cd ~/hpb-educ/lab1/nodejs
buildah from --name todobuild node:5
buildah config --workingdir /home/node/app todobuild
buildah copy todobuild . /home/node/app
buildah run todobuild npm install
buildah config --cmd '["node","app.js"]' todobuild
buildah commit todobuild todo:latest
```

2. List the images with podman.

```podman ps```

3. Start the todo app like in task 2.

```podman run -d -p 30080 -e MYSQL_ENV_MYSQL_DATABASE=todo -e MYSQL_ENV_MYSQL_USER=todo -e MYSQL_ENV_MYSQL_PASSWORD=todopassw0rd --name todo todo:latest```

4. Observe the logs of todo app.

```podman logs todo```

5. Access the application through URL.

```podman ps```

Note the port and open URL ```http://hpb1.tn.hr:<port>/todo/```

6. Stop and cleanup everything.

```
podman stop todo
podman rm -af
podman rmi -af
```