# Project Documentation: Dockerizing a Rails Application with Nginx and MySQL

Helloo, this is Siddanth! I'm going to guide you how I was able to setup Rails application in a Docker environment, step by step. This setup includes running our app with a separate MySQL database container, using Nginx as a reverse proxy, adding persistence, managing containers with Docker Compose, and implementing rate limiting with Nginx.

## 1. Dockerizing the Rails Application

Firstly, I had to encapsulate our application within a Docker container. This process involved creating a Dockerfile that specifies how the Rails environment is set up, including installing necessary dependencies and copying the application code into the container. The goal was to create an isolated environment that mimics our production setup, ensuring consistency across development, testing, and production. 

Here's the DockerFile : *******

Build the image with:

```
docker build -t my-rails-app .
```



