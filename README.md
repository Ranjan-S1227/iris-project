# Project Documentation: Dockerizing a Rails Application with Nginx and MySQL

Hello, this is Siddanth, pursuing Civil Engg. (I year) ! I'm going to guide you how I was able to setup Rails application in a Docker environment, step by step. This setup includes running our app with a separate MySQL database container, using Nginx as a reverse proxy, adding persistence, managing containers with Docker Compose, and implementing rate limiting with Nginx.

## Flow of the application 

![IMG_5446](https://github.com/Ranjan-S1227/iris-project/assets/76945354/adffd6f0-546c-4b89-b3a6-73fffa237e30)


## 1. Dockerizing the Rails Application

Firstly, I had to encapsulate our application within a Docker container. This process involved creating a **Dockerfile** that specifies how the Rails environment is set up, including installing necessary dependencies and copying the application code into the container. The goal was to create an isolated environment that mimics our production setup, ensuring consistency across development, testing, and production. 

Here's the DockerFile : [DockerFile](https://github.com/Siddanth-S/iris-project/blob/main/Dockerfile)
which basically describes installation of all the dependencies, copies the application code into the image and set the entry point. 

Build the image with the DockerFile:

```yml
docker build -t my-rails-app .
```
![Screenshot 2024-03-17 at 11 50 51 PM](https://github.com/Ranjan-S1227/iris-project/assets/76945354/74ec83ea-cbda-44eb-b5fe-fc3bad4f27e9)


But in this it's not necessary to run this command as running docker-compose.yml is sufficient to run DockerFile and create the application image. 


## 2. Launching the Application with a Database Container

I introduced a MySQL container to act as database. Pulled the mysql:latest image from the Docker Hub. Then configured the mysql environmental variables as below inside the [docker-compose.yml](https://github.com/Siddanth-S/iris-project/blob/main/docker-compose.yml). By leveraging Docker Compose, I orchestrated the simultaneous launch of the application and database container. A significant consideration was ensuring the database port was not exposed externally, mitigating potential security risks. This setup also allowed us to expose our application on port 8080, enabling access from "localhost:8080".

``` 
docker-compose up  #to launch the containers simultaneously
```
![Screenshot 2024-03-17 at 11 54 34 PM](https://github.com/Ranjan-S1227/iris-project/assets/76945354/5718265a-88f3-49d5-b1ae-3f51090f6d75)

Connection Successful !! 
 ![Screenshot 2024-03-18 at 12 24 38 AM](https://github.com/Ranjan-S1227/iris-project/assets/76945354/b0fc3600-c98a-4055-b06e-2ef4cc309f75)


## 3. Introducing Nginx as a Reverse Proxy

I integrated an Nginx container to serve as a reverse proxy for the application's accessibility and security. This setup directed incoming traffic through Nginx before reaching the Rails application, providing an additional layer of abstraction and security. Configuring Nginx for reverse proxying also allowed us to hide the inner workings of our application network. 

Here's the Nginx config : [Nginx Conf](https://github.com/Siddanth-S/iris-project/blob/main/nginx/nginx.conf)


## 4.  Scaling with Multiple Application Containers

I setup to include two additional Rails containers, ensuring our application could handle increased traffic gracefully. This scaling was facilitated by Nginx's load balancing feature, which distributed incoming requests evenly across the three containers. This approach improved application's availability. 
![Screenshot 2024-03-17 at 11 57 17 PM](https://github.com/Ranjan-S1227/iris-project/assets/76945354/5a573228-bd8c-4a03-a2f6-e257d1dffb72)


## 5.  Persisting Data and Configurations

Through Docker volumes, achieved persistence by mapping specific directories from our containers to the host machine. This ensured that vital data and configurations remained intact even if the containers were restarted or removed, providing a reliable and consistent experience.
Here's the implemtation : [Volume data persistance](https://github.com/Siddanth-S/iris-project/blob/97d00c77704391e458c2b78b487de303e982bf0f/docker-compose.yml#L73)

## 6. Simplifying Deployment with Docker Compose

To streamline the deployment process and ensure all components of the application ecosystem could be launched with a single command, utilized Docker Compose. This tool allowed me to define and manage multi-container Docker applications, simplifying complex deployments into manageable configurations.

Here's the compose file : [docker-compose.yml](https://github.com/Siddanth-S/iris-project/blob/main/docker-compose.yml)


## 7. Implementing Rate Limiting with Nginx

Implementing rate limiting through Nginx is a strategy to prevent potential denial-of-service (DoS) attacks caused by an overwhelming number of requests. By setting thresholds for the number of allowed requests per second ,  ensured the application remained available to legitimate users.
```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;
```
Defines a zone named mylimit and allocates 10 megabytes of memory to store the states of clients' addresses. rate=5r/s: Specifies the allowed request rate, which is 5 requests per second in this case.
Tested it out with ***Apache Bench*** (ab)  : ``` ab -n 100 -c 10 http://localhost:8080/ ```
Here's the result : 
```
Benchmarking localhost (be patient).....done


Server Software:        nginx/1.25.4
Server Hostname:        localhost
Server Port:            8080

Document Path:          /
Document Length:        3730 bytes

Concurrency Level:      10
Time taken for tests:   7.695 seconds
Complete requests:      100
Failed requests:        98
   (Connect: 0, Receive: 0, Length: 98, Exceptions: 0)
Non-2xx responses:      67
Total transferred:      211247 bytes
HTML transferred:       140699 bytes
Requests per second:    13.00 [#/sec] (mean)
Time per request:       769.484 [ms] (mean)
Time per request:       76.948 [ms] (mean, across all concurrent requests)
Transfer rate:          26.81 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       0
Processing:     1  498 1032.0      5    4145
Waiting:        1  498 1031.4      5    4144
Total:          1  499 1032.0      5    4145

Percentage of the requests served within a certain time (ms)
  50%      5
  66%     20
  75%    268
  80%    848
  90%   2277
  95%   2911
  98%   4089
  99%   4145
 100%   4145 (longest request)

```
Observations from Benchmark
Non-2xx Responses: You observed 67 non-2xx responses out of 100 total requests, which indicates that some requests were either queued (delayed) or rejected due to exceeding the rate limit.
Failed Requests: The presence of failed requests due to content length mismatches likely reflects varied responses, such as HTTP 503 Service Temporarily Unavailable status codes, which are served when requests exceed your configured rate limit and burst capacity.


Indications of Active Rate Limiting
The direct cause-and-effect relationship where the rate of requests (as generated by your benchmarking tool) exceeds the configured limit (5r/s) and results in a notable number of non-2xx responses and request failures due to Nginx enforcing these limits.
The variability in response times and the existence of requests that took significantly longer to process (max time of 4145 ms) can also be indicative of rate limiting in action, as requests that exceed the limit might be queued, contributing to longer processing times.


These observations collectively indicate that the rate limiting setup is active and effectively limiting the rate at which individual clients can make requests to  server, as per the configuration you've established.


## Result 

All the tasks(except Bonus) are completed and the application is perfectly running in the local(development) environment. 
![Screenshot 2024-03-18 at 12 08 32 AM](https://github.com/Ranjan-S1227/iris-project/assets/76945354/3247dbad-9934-4355-a847-d16dbe046129)


![Screenshot 2024-03-18 at 12 09 08 AM](https://github.com/Ranjan-S1227/iris-project/assets/76945354/0f159f91-968c-4b01-bb9f-ff228aa80605)



