# Containerized VProfile Web Application Project

## Overview
<div style="text-align: center;">
The objective of the "Containerized Vprofile Web App Project," based on code obtained from the Udemy course by Imran Teli, is to create a highly scalable and modular web application using Java, Maven, Tomcat, and Docker. This project aims to empower users to efficiently create, manage, and present their professional and personal profiles while incorporating technologies like MySQL, Memcached, and RabbitMQ for secure and optimized data storage and messaging within containerized environments. By acknowledging the code's origin from Imran Teli's course, I aim to build a user-friendly and technologically advanced platform for profile management, and personal branding.
</div>

## Features

- Built using Java and Maven.
- Utilizes the Tomcat web server for application deployment.
- Manages data storage through a MySQL database.
- Implements Redis for caching and data retrieval optimization.
- Used Memcached to cache the database
- Utilizes RabbitMQ for asynchronous messaging.

## Project Architecture
<div style="text-align: center;">
The VPro web application project employs Java, Maven, MySQL, Memcached, RabbitMQ, and Tomcat for its architecture. Java forms the core of the application's logic, while Maven streamlines the development process. MySQL handles data storage, and Memcached optimizes data retrieval. RabbitMQ facilitates asynchronous communication, enhancing the application's responsiveness. Tomcat serves as the web server, handling HTTP requests.

To ensure consistent and manageable deployment, the project utilizes Docker. Three Dockerfiles are created for web, app, and DB components, allowing for isolated and efficient containerization. Docker Compose orchestrates these containers for streamlined management. Once the project is successfully deployed, it's pushed to DockerHub, ensuring easy future use and distribution across different environments. This architecture emphasizes performance, scalability, and simplicity in deployment.
</div>
- **GitHub Repository**: [https://github.com/Arup-gope/vprofile-project]
  
![Architecture](https://github.com/Arup-gope/containerized-vpro-web/assets/64405321/67622a1c-a278-4c55-b413-e5fb1b94bf29)
                      Fig. 1: Architecture of the project.

## How to Run

To run this project locally, follow these steps:

1. Pull the project's docker images from the Docker Hub using the given link.

2. Execute the Docker Compose file to bring the project up.

3. Access the application in  the web browser at [http://localhost:8080](http://localhost:8080)


## Docker Images & Docker Compose File
### Docker Images

We have published the following Docker images for the VProfile Project on Docker Hub:

- **vprofiledb**: The Docker image for the database component of the project.
  - Docker Hub Repository: [arupgope/vprofiledb](https://hub.docker.com/r/arupgope/vprofiledb)
  - Docker Pull Command: `docker pull arupgope/vprofiledb`

- **vprofileapp**: The Docker image for the application component of the project.
  - Docker Hub Repository: [arupgope/vprofileapp](https://hub.docker.com/r/arupgope/vprofileapp)
  - Docker Pull Command: `docker pull arupgope/vprofileapp`

- **vprofileweb**: The Docker image for the web component of the project.
  - Docker Hub Repository: [arupgope/vprofileweb](https://hub.docker.com/r/arupgope/vprofileweb)
  - Docker Pull Command: `docker pull arupgope/vprofileweb

We can use the provided Docker images to deploy and run the VProfile Project in a containerized environment. This Dockerfile is used to build and deploy the web application. It starts by setting up a build image using the OpenJDK 11 base image and installs Maven for the build process. It then clones the VProfile project repository from GitHub, specifically the "docker" branch, and builds the project using Maven.

After the build phase, it switches to the Tomcat 9 with Java 11 base image for the application runtime. The existing web applications in Tomcat are removed to ensure a clean slate.

The VProfile web application (in the form of a WAR file) generated in the build image is copied to the Tomcat webapps directory, where it will be deployed as the ROOT application.

Port 8080 is exposed to allow external access to the application, and the CMD command specifies running Tomcat using the "catalina.sh run" command, starting the application within the container.

Overall, this Dockerfile sets up the environment for building and deploying the VProfile web application using Maven and Tomcat, making it accessible on port 8080 in the final container.
~~~
FROM openjdk:11 AS BUILD_IMAGE
RUN apt update && apt install maven -y
RUN git clone https://github.com/devopshydclub/vprofile-project.git
RUN cd vprofile-project && git checkout docker && mvn install 

FROM tomcat:9-jre11
LABEL "Project"="Vprofile"
LABEL "Author"="Gope"
RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
~~~
This Dockerfile builds a MySQL container with version 8.0.33, setting labels for the project and author. It configures environment variables, including a root password, and creates a database named "accounts." Additionally, it adds an SQL backup file to the container, which is automatically executed upon container initialization, populating the "accounts" database with data from "db_backup.sql."
~~~
FROM mysql:8.0.33
LABEL "Project"="Vprofile"
LABEL "Author"="Gope"

ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"

ADD db_backup.sql /docker-entrypoint-initdb.d/db_backup.sql

~~~

This Dockerfile uses the official Nginx image as a base. It sets project and author labels for identification. It removes the default Nginx configuration file and replaces it with a custom configuration file, "nginvproapp.conf," located in the /etc/nginx/conf.d/ directory, allowing for custom Nginx configuration for the "Vprofile" project.

~~~
FROM nginx
LABEL "Project"="Vprofile"
LABEL "Author"="Gope"

RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginvproapp.conf /etc/nginx/conf.d/vproapp.conf

~~~
### Docker Compose File
This Docker Compose configuration file, using version 3.8, orchestrates a multi-container application setup. It includes services for a MySQL database, Redis, Memcached, RabbitMQ, an application container, and a web server. The "vprodb" service builds a MySQL container, forwarding port 3306, and configuring data persistence with the "vprodbdata" volume. "redis" uses an official Redis image, "vprocache01" deploys Memcached, and "vpromq01" utilizes RabbitMQ, all with specific port configurations. The "vproapp" service builds an application container, exposing port 8080 and using the "vproappdata" volume for data storage. Lastly, the "vproweb" service creates a web server container, forwarding port 80. These containers work together to provide the foundation for the VProfile application's various components, ensuring scalability and modularity in a Dockerized environment.

~~~
version: '3.8'
services:
  vprodb:
    build: 
      context: ./Docker-files/db
    image: arupgope/vprofiledb
    container_name: vprodb
    ports:
      - "3306:3306"
    volumes:
      - vprodbdata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=vprodbpass
  redis:
    image: "redis:alpine"

  vprocache01:
    image: memcached
    ports:
      - "11211:11211"

  vpromq01:
    image: rabbitmq
    ports:
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest


  vproapp:
    build: 
      context: ./Docker-files/app
    image: arupgope/vprofileapp
    container_name: vproapp
    ports:
      - "8080:8080"
    volumes:
      - vproappdata:/usr/local/tomcat/webapps


  vproweb:
    build: 
      context: ./Docker-files/web
    image: arupgope/vprofileweb
    container_name: vproweb
    ports:
      - "80:80"


volumes:
  vprodbdata: {}
  vproappdata: {}

~~~

## Result
After I access the website at http://localhost:8080, I encounter a login page. To access the site, I need to log in with the following credentials: UserID: admin_vp, and password: admin_vp. Upon logging in, I am directed to the front page, as shown in Figure 2.
![Front_Page](https://github.com/Arup-gope/containerized-vpro-web/assets/64405321/535d6343-b96c-492b-b1e1-51e1bedd3181)
Fig. 2: Login Page of this website.

Once I've successfully logged in, I have access to a list of all user IDs. Clicking on a specific User ID provides me with detailed information about that particular user. This information is retrieved from a MYSQL database and cached. When I revisit the website in the future, the information will be fetched from Memcached.


![MySql](https://github.com/Arup-gope/containerized-vpro-web/assets/64405321/a5faecde-8701-49bf-9962-f3b824a7c422)


Fig. 3: User's Data from MYSQL


![Memcached_Checked](https://github.com/Arup-gope/containerized-vpro-web/assets/64405321/fa63dcaf-f210-4aa2-9e66-0ad7256b35a4)
Fig. 4: User's Data from Memchached.


## Docker Image Push Report

I have successfully pushed the following Docker images for the VProfile Project to Docker Hub:

- **vprofiledb**: The Docker image for the database component of the project.
  - Docker Hub Repository: [arupgope/vprofiledb](https://hub.docker.com/r/arupgope/vprofiledb)
  - Docker Push Command: `docker push arupgope/vprofiledb`

- **vprofileapp**: The Docker image for the application component of the project.
  - Docker Hub Repository: [arupgope/vprofileapp](https://hub.docker.com/r/arupgope/vprofileapp)
  - Docker Push Command: `docker push arupgope/vprofileapp`

- **vprofileweb**: The Docker image for the web component of the project.
  - Docker Hub Repository: [arupgope/vprofileweb](https://hub.docker.com/r/arupgope/vprofileweb)
  - Docker Push Command: `docker push arupgope/vprofileweb`


...

