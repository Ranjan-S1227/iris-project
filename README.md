# Project Documentation: Dockerizing a Rails Application with Nginx and MySQL

Helloo, this is Siddanth! I'm going to guide you how I was able to setup Rails application in a Docker environment, step by step. This setup includes running our app with a separate MySQL database container, using Nginx as a reverse proxy, adding persistence, managing containers with Docker Compose, and implementing rate limiting with Nginx.

## 1. Dockerizing the Rails Application

Firstly, I had to encapsulate our application within a Docker container. This process involved creating a Dockerfile that specifies how the Rails environment is set up, including installing necessary dependencies and copying the application code into the container. The goal was to create an isolated environment that mimics our production setup, ensuring consistency across development, testing, and production. 

Here's the DockerFile : *******

Build the image with:

```yml
docker build -t my-rails-app .
```

## 2. Launching the Application with a Database Container

I introduced a MySQL container to act as database. Pulled the mysql:latest image from the Docker Hub. Then configured the mysql environmental variables as below inside the docker-compose.yml. By leveraging Docker Compose, I orchestrated the simultaneous launch of the application and database container. A significant consideration was ensuring the database port was not exposed externally, mitigating potential security risks. This setup also allowed us to expose our application on port 8080, enabling access from "localhost:8080".

``` 
docker-compose up # to launch the containers simultaneously
```

## 3. Introducing Nginx as a Reverse Proxy

I integrated an Nginx container to serve as a reverse proxy for the application's accessibility and security. This setup directed incoming traffic through Nginx before reaching the Rails application, providing an additional layer of abstraction and security. Configuring Nginx for reverse proxying also allowed us to hide the inner workings of our application network. 








