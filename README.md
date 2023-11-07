# Question 1.1

This would be the image : 
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
```

and you add these commands to get your image settings in docker :
```
docker pull postgres
docker pull adminer
docker run -p "8090:8080" --network app-network --name=adminer  -d adminer 
docker run -d --network app-network --name db mydb     
```

We add 2 lines in the Dockerfile to insert the database :
```
COPY CreateScheme.sql /docker-entrypoint-initdb.d
COPY InsertData.sql /docker-entrypoint-initdb.d
```
# Question 1.2

Multi-stage builds are useful to anyone who has struggled to optimize Dockerfiles while keeping them easy to read and maintain.

It allows devs to swiftly create temporary Docker containers for third-party applications, databases and webservices and other components.
This file is a Dockerfile, which is a text document that contains all the commands a user could call on the command line to assemble an image. In this specific Dockerfile, there are two main sections, 'Build' and 'Run'.

## Build Section:
The first section starts with ```FROM maven:3.8.6-amazoncorretto-17 AS myapp-build```, which specifies the base image to use for the build phase, in this case, it's using the ```maven:3.8.6-amazoncorretto-17``` image. This image is used to build the application using Maven.

    - ENV MYAPP_HOME /opt/myapp sets the environment variable MYAPP_HOME to '/opt/myapp'.
    
    - WORKDIR $MYAPP_HOME sets the working directory for the subsequent commands to '/opt/myapp'.
    
    - COPY pom.xml . and COPY src ./src copies the project's pom.xml file and the src directory into the container's working directory.
    
    - RUN mvn package -DskipTests runs the mvn package command with the option -DskipTests to skip running tests during the build process.

## Run Section:
The second section begins with ```FROM amazoncorretto:17```, specifying the base image to use for running the application.

    - ENV MYAPP_HOME /opt/myapp sets the environment variable MYAPP_HOME to '/opt/myapp'.

    - WORKDIR $MYAPP_HOME sets the working directory for the subsequent commands to '/opt/myapp'.

    - COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar copies the built JAR file from the build stage to the working directory in the current stage.

    - ENTRYPOINT java -jar myapp.jar sets the entry point command that will be executed when the Docker container is run. In this case, it runs the Java application contained in the myapp.jar file using the java -jar command.

# Question 1.3

Here are some of the most important commands used with Docker Compose:

````
docker-compose up: 
Builds, (re)creates, starts, and attaches to containers for a service. It creates and starts all the containers defined in the docker-compose.yml file.

docker-compose down: 
Stops and removes containers, networks, volumes, and images created by docker-compose up.

docker-compose build: 
Builds or rebuilds services defined in the docker-compose.yml file.

docker-compose pull: 
Pulls images for services defined in the docker-compose.yml file.

docker-compose stop: 
Stops running containers without removing them. They can be started again with docker-compose start.

docker-compose start: 
Starts existing containers that were stopped.
````

# Question 1.4

````
version: '3.7'

services:
    backend:
      container_name: back_end
      build: .\simple-api-student-main\
      networks:
        - app-network
      depends_on:
        - database

    database:
      container_name: db
      build: .\TP_Docker\
      networks:
        - app-network

    httpd:
      container_name: front_end
      build: .\TP_Docker_HTTP\
      ports:
        - "8080:80"
      networks:
        - app-network
      depends_on:
        - backend

networks:
  app-network:
````

## Version: 
This file is written for version 3.7 of the Compose file format.

## Services:

### backend:

- ```container_name``` specifies the name of the container as "back_end".
- ```build``` specifies the path to the Dockerfile or the build context.
- ```networks``` attaches the service to the "app-network" network.
- ```depends_on``` ensures that the "backend" service starts after the "database" service has started.

### database:

- ```container_name``` sets the name of the container as "db".
- ```build``` points to the path of the Dockerfile or the build context.
- ```networks``` attaches the service to the "app-network" network.

### httpd:

- ```container_name``` defines the name of the container as "front_end".
- ```build``` specifies the path to the Dockerfile or the build context.
- ```ports``` maps the host's port 8080 to the container's port 80.
- ```networks``` attaches the service to the "app-network" network.
- ```depends_on``` ensures that the "httpd" service starts after the "backend" service has started.

## Networks:
```app-network``` defines the network named "app-network" that all the services are connected to. If not explicitly defined, Docker Compose creates a default network for the services.

# Question 1.5

We first tag our image. For now, we have been only using the latest tag, now that we want to publish it, let’s add some meaningful version information to our images.
To do this we use :

    docker tag my-database USERNAME/my-database:1.0
We can then publish it :

    docker push USERNAME/my-database:1.0  

# Question 2.1

Testcontainers is a Java library that supports writing integration tests by providing lightweight, throwaway instances of common databases, web browsers, or anything else that can run in a Docker container.

# Question 2.2

    name: CI devops 2023
    on:
    #to begin you want to launch this job in main and develop
    push:
    branches: master
    pull_request:
    
    jobs:
    test-backend:
    runs-on: ubuntu-22.04
    steps:
    #checkout your github code using actions/checkout@v2.5.0
    - uses: actions/checkout@v2.5.0
    
         #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
          - name: Set up JDK 17
            uses: actions/setup-java@v3
            with:
              java-version: '17'
              distribution: 'adopt'
              username: ${{ secrets.DOCKERHUB_USERNAME }}
              password: ${{ secrets.DOCKERHUB_TOKEN }}


## Name: 
CI devops 2023 - It is the name of the GitHub Actions workflow.

## On section: 
This section specifies the triggering events for the workflow.

- ```push```: Triggers the workflow when changes are pushed to the branches specified in the branches field.
- ```pull_request```: Triggers the workflow when a pull request is opened or updated.

## Jobs section: 
This section defines the different jobs that the workflow will perform.

### test-backend:
#### runs-on: 
Specifies the operating system environment where the job will run, in this case, it is set to ```ubuntu-22.04```.
##### Steps:
- Checkout GitHub code using the ```actions/checkout@v2.5.0``` action.
- Set up JDK 17 using the ```actions/setup-java@v3``` action to configure the Java environment, with the specified Java version and distribution. The Docker Hub username and token are fetched from the GitHub secrets.


# Question 2.3

    #finally build your app with the latest command
    - name: Build and test with Maven
    run: mvn -B verify sonar:sonar -Dsonar.projectKey=florianguyot91_TP_Docker -Dsonar.organization=florianguyot91 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./simple-api-student-main/pom.xml

## Build and test with Maven: 
Runs the Maven command to build and test the application. It includes the verify goal and SonarQube analysis. The ```sonar.projectKey```, ```sonar.organization```, ```sonar.host.url```, and ```sonar.login``` parameters are set for the SonarCloud integration. The build is performed using the ```pom.xml``` file located in the ```simple-api-student-main``` directory

# Question 3.1

    all:
    vars:
    ansible_user: centos
    ansible_ssh_private_key_file: /home/florian/id_rsa
    children:
    prod:
    hosts: florian.guyot.takima.cloud


## Inventory File ```inventories/setup.yml```:

- The file organizes the hosts into a group called prod.
- It defines variables under vars, including ```ansible_user``` and ```ansible_ssh_private_key_file```.
- The ```ansible_user``` variable specifies the remote user to use for the connection.
- The ```ansible_ssh_private_key_file``` variable specifies the path to the private key file for SSH authentication.


    ansible all -i inventories/setup.yml -m ping


## Test Inventory with Ping Command:

- This command tests the connectivity to the hosts defined in the inventory file by using the ping module.
- The ```-i``` flag specifies the inventory file.
- The ```-m``` flag specifies the module to use, in this case, ping.    



    ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

## Check System Information with Setup Module:

- This command retrieves setup facts from the hosts using the setup module.
- The ```-a``` flag provides additional parameters to the module. In this case, the filter parameter filters the setup facts to those related to the distribution.


    ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become

## Uninstall Apache HTTP Server with Yum Module:

- This command removes the Apache HTTP Server from the target machines using the ```yum``` module.
- The ```-a``` flag provides additional parameters to the module. In this case, ```name=httpd``` specifies the package name to remove, and ```state=absent``` indicates the desired state.
- The ```--become``` flag is used to execute the command with escalated privileges.


# Question 3.2



## Hosts: 
The playbook targets all hosts specified in the Ansible inventory.

## ```gather_facts```: 
This parameter is set to false, which means Ansible will not gather facts about the target hosts.

## ```become```: 
This parameter is set to true, indicating that the tasks will run with escalated privileges.

## Tasks:

- Install ```device-mapper-persistent-data```: Installs the ```device-mapper-persistent-data``` package using the ```yum``` package manager.
- Install ```lvm2```: Installs the ```lvm2``` package using the yum package manager.
- Add Docker repository: Adds the Docker repository using the ```yum-config-manager``` command.
- Install Docker: Installs the ```docker-ce``` package using the ```yum``` package manager.
- Install python3: Installs the python3 package using the yum package manager.
- Install Docker Python library: Installs the docker Python library using ```pip3```. The ansible_python_interpreter variable is set to ensure that the correct Python version is used.
- Ensure Docker is running: Starts the Docker service using the service module.

### Question 3.3 :
In this section, we modified the file playbook.yml to add roles for each task (docker, network, database, app, and proxy):
 
    - name: Deploy My Application
      hosts: all
      gather_facts: false
      become: yes
      roles:
        - docker
        - network
        - database
        - app
        - proxy

We then created a directory roles with packages for each role.
For the docker role, the content of the file allows for the automatic installation of Docker:

    - name: Install device-mapper-persistent-data
      yum:
      name: device-mapper-persistent-data
      state: latest
    
    - name: Install lvm2
        yum:
        name: lvm2
        state: latest
    
    - name: add repo docker
          command:
          cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    
    - name: Install Docker
          yum:
          name: docker-ce
          state: present
    
    - name: Make sure Docker is running
          service: name=docker state=started
          tags: docker

For the network role, the following code creates the network used by Docker:


    - name: Create Docker Network
      docker_network:
      name: app-network
      state: present

For the database role, the following code runs the database, the corresponding Docker container, and creates the corresponding image:


    - name: Run database
      docker_container:
      name: db
      image: florian007/database:1.0
      networks:
      - name: app-network

For the app role, the following code runs the backend, the corresponding container, and the image:


    - name: Run backend
      docker_container:
      name: my-running-app
      image: florian007/simple-api-student-main:latest
      networks:
      - name: app-network

For the proxy role, the following code runs the frontend on port 80, the corresponding container, and the image:

    - name: Run Proxy Container
      docker_container:
      name: my-front-app
      ports:
      - "80:80"
      image: florian007/http:1.0
      networks:
      - name: app-network

