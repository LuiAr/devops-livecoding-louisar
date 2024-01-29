
# **Report - DevOps course by Takima**
*made by Louis Arbey*

---
## $ First Part - Docker

As Docker has already been done a lot in recent classes we didn't took a lot of time on this part.
But I'm still gonna give answers for the *Questions* part.

***1-1 Document your database container essentials: commands and Dockerfile.***

- `Dockerfile`
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

Commands
- `docker build -t my-database .`: Build the image
- `docker run -d --name my-postgres-db -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -p 5432:5432 my-database`: Run the container
- `docker ps`: List all running containers

***1-2 Why do we need a multistage build? And explain each step of this dockerfile.***

-> We need a multistage build as it optimize build speed & size. It also allows to keep the runtime environment minimal ans secure.

***1-3 Document docker-compose most important commands.***

- `docker-compose up`: Starts and runs the app
- `docker-compose down`: Stops and removes containers, etc... created
- `docker-compose build`: Builds or rebuilds services
- `docker-compose logs`: View output

***1-4 Document your docker-compose file.***

```
version: '3.7'

services:
  backend:
    build: ./path/to/backend/Dockerfile
    networks:
      - my-network
    depends_on:
      - database

  database:
    image: my-database
    networks:
      - my-network

  httpd:
    build: ./path/to/httpd/Dockerfile
    ports:
      - "80:80"
    networks:
      - my-network
    depends_on:
      - backend

networks:
  my-network:
```

***1-5 Document your publication commands and published images in dockerhub.***

- `docker login`: To login to dockerhub
- `docker tag my-db luiar/my-db`: Tag the image
- `docker push luiar/my-db`: Push the image to dockerhub

---
## $ Second Part - Github Actions

I had a lot of problems when running `mvn clean verify` in the sample project.
In the end the two reasons of the problems where:
- Not the right version of java sdk
- Docker wasn't running properly in the background

***2-1 What are testcontainers ?***

-> Testcontainers are Java libraries that allow running Docker containers for testing

***2-2 Document your Github Actions configurations.***

- `main.yml`
```
- branches: [ main ]
- name: Set up JDK 17
    uses: actions/setup-java@v3
    with:
      java-version: '17'
      distribution: 'temurin'
- name: Build and test with Maven 
  run: mvn clean install --file simple-api/pom.xml
```

---
## $ Third Part - Ansible

***3-1 Document your inventory and base commands***

- `/ansible/inventories/setup.yml`

This file is used to define the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.
```
  all:
  vars:
  ansible_user: centos
  ansible_ssh_private_key_file: "/Users/louisar/Library/CloudStorage/OneDrive-Efrei/! COURS M2/Application of Big Data/takima/id_rsa"
  children:
  prod:
  hosts:
  - centos@louis.arbey.takima.cloud
```

- `ansible all -i /Users/louisar/ansible_hosts.txt -i /Users/louisar/ansible/inventories/setup.yml -m ping`

This command is used to ping all the hosts in the inventory file.

- `ansible all -i /Users/louisar/ansible_hosts.txt -i /Users/louisar/ansible/inventories/setup.yml -m setup -a "filter=ansible_distribution*" --private-key="/Users/louisar/Library/CloudStorage/OneDrive-Efrei/! COURS M2/Application of Big Data/takima/id_rsa"`

This command is used to get the distribution of all the hosts in the inventory file.

***3-2 Document your playbook***

- `install_docker.yml`: This playbook is used to install docker on the host.

```
  ---
  - hosts: all
  gather_facts: false
  become: true 

  roles:
    - docker

  tasks:
    - name: Install device-mapper-persistent-data
      yum:
      name: device-mapper-persistent-data
      state: latest

    - name: Install lvm2
      yum:
      name: lvm2
      state: latest

    - name: Add Docker repository
      command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install Docker
      yum:
      name: docker-ce
      state: present

    - name: Install python3
      yum:
      name: python3
      state: present

    - name: Install docker Python module with Python 3
      pip:
      name: docker
      executable: pip3
      vars:
      ansible_python_interpreter: /usr/bin/python3

    - name: Make sure Docker is running
      service:
      name: docker
      state: started
      tags: docker
```

### **Deploy your App**

***Document your docker_container tasks configuration.***

- `Database`

```
--- tasks file for roles/launch_database

- name: Run PostgreSQL Container
  docker_container:
  name: my-db
  image: luiar/database-image
  networks:
  - name: my-network
```

- `API`
```
--- tasks file for roles/launch_app


- name: Run Simple-API Application Container
  docker_container:
  name: my-api
  image: luiar/tp-devops-simple-api
  networks:
  - name: my-network
```

- `Proxy`
```
--- tasks file for roles/launch_proxy

- name: Run HTTPD Container
  docker_container:
  name: my_httpd
  image: luiar/httpd-image
  published_ports:
  - "80:80"
  networks:
  - name: my-network
```

- `Network`
```
--- tasks file for roles/create_network

- name: Create a Docker network
  community.docker.docker_network:
  name: my-network
```

I have to add to this part that it took so long to do it because of one reason.
> The network of Efrei Paris was blocking us from accessing `https://louis.arbey.takima.cloud/` (it also seems that accessing it with http was working better in general)

That's why I didn't have time *and motivation* to finish the last part of the TP.

# That's a wrap, thank you for reading !