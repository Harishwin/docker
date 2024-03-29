# docker
scenario = if want to make 50 containers with diff. specification we can use docker compose.
condition = docker should install.
          = compose should install.
go to google or ```sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```

Provide execute permissions to the Docker Compose binary: ```sudo chmod +x /usr/local/bin/docker-compose```

-------------------------------------------------------------------------------------------------------------------------------------------------------------------
## scenario = wordpress & db using docker compose.
make yaml file = ```vim docker-compose.yml```

## use wordpress deployment syntax using docker documentation :
services:
  db:
    We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    If you really want to use MySQL, uncomment the following line
    image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
  wp_data:


------------------------------------------------------------------------
Now, run ```docker compose up -d``` from your project directory
```docker container ls```

--------------------------------------------------------------------------
## scenario: Using compose we can create multiple app conatiners,
create new yml or json file , 
for run new file use following command,
```docker-compose -f your_compose_file.yml up -d```
```vim multiple.yml```

version: '3'
services:
  web1:
    container_name: multi1
    image: nginx:alpine  # Use the nginx image
    ports:
      - "8080:80"  # Map host port 8080 to container port 80

  web2:
    container_name: multi2
    image: nginx:alpine  # Use the nginx image
    ports:
      - "8081:80"  # Map host port 8081 to container port 80

command : ```docker-compose -f multiple.yml up -d```
----------------------------------------------------------------------------------
## scenario: need multiple copies of exiting conatiners? use following command
```docker-compose -f multiple.yml up -d --scale web1=3```
```docker-compose -f multiple.yml down```

-----------------------------------------------------------------------------------
## DOCKER NETWORK:
# scenario: docker host can communicate with bridge network.
if we outside try to hit inside. if we map inner conatiner using port mapping then its possible.
Create a Custom Bridge Network:
docker network create my_network
# Run Containers and Connect to the Custom Network:
```docker run -d --name container1 --network my_network nginx```
```docker run -d --name container2 --network my_network nginx```
Here, we've run two containers named container1 and container2, connecting them to the my_network custom bridge network.

# Verify Connectivity:

Inside container1, you can ping container2 by its container name:
```docker exec -it container1 ping container2```
This demonstrates that containers on the same custom network can communicate using container names.

By using Docker networks, you can isolate containers, control communication between them, and connect containers to external networks or other containers based on your application's needs.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## scene: How to communicate two diff network conatiner each other.
# Connect Containers to the Same Docker Network:
The simplest approach is to connect both containers to the same Docker network. This allows them to communicate directly with each other using the network alias or IP address within the shared Docker network.
# Create a custom Docker network
```docker network create my-network```

# Run container1 and connect to the custom network
```docker run -d --network=my-network --name container1 your_image1```

# Run container2 and connect to the same custom network
```docker run -d --network=my-network --name container2 your_image2```
Use Docker Compose:
Docker Compose allows you to define and manage multi-container applications. You can specify networks and services in a ```docker-compose.yml``` file, making it easier to manage complex setups.

version: '3'
services:
  container1:
    image: your_image1
    networks:
      - my-network

  container2:
    image: your_image2
    networks:
      - my-network

networks:
  my-network:
    driver: bridge
Then run ```docker-compose up -d``` to start the services.

# Use Docker Bridge Networking:
Docker bridge networking allows containers to communicate with each other if they are on the same bridge network. However, this approach usually requires manual configuration and is less portable than using custom networks or Docker Compose.

# Use Docker Swarm:
If you're using Docker Swarm, you can create overlay networks that span across multiple hosts, enabling communication between containers running on different hosts.
```docker network create --driver=overlay my-overlay-network```
Then, when creating services in the Swarm, you can specify the network to use:

```docker service create --name service1 --network my-overlay-network your_image1```
```docker service create --name service2 --network my-overlay-network your_image2```

Choose the option that best fits your use case and networking requirements. Each approach has its own advantages and trade-offs based on your specific scenario and infrastructure setup.
