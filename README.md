# RedisBank - A Simple Mobile Banking Application 

![My Image](https://github.com/redis-developer/redisbank/blob/main/redisbank2.png)

Redisbank is a real-time personal finance management application built using Spring Boot - an open source, microservice-based Java web framework. The Spring Boot framework creates a fully production-ready environment that is completely configurable using its prebuilt code within its codebase. The application shows a searchable transaction overview with realtime updates as well as a personal finance management overview with realtime balance and biggest spenders updates.

### Tech Stack:

- Java
- Spring Boot
- Bootstrap
- CSS
- Vue

The application uses the below Redis core-data structure and Modules:


- Redis Streams for the realtime transactions
- RedisTimeSeries for the balance over time
- RediSearch for searching transactions
- Sorted Sets for the 'biggest spenders'
- Redis hashes for session storage (via Spring Session)

# Architecture
<img src="architecture.png"/>

# Getting Started

## Prerequisites

1. JDK 11 or higher (https://openjdk.java.net/install/index.html)
2. Docker Desktop (https://www.docker.com/products/docker-desktop)
3. Azure CLI (https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
4. Azure Spring Cloud extension for the Azure CLI (https://docs.microsoft.com/en-us/cli/azure/spring-cloud?view=azure-cli-latest)

## How to run it locally

1. Clone the repository

```
git clone https://github.com/redis-developer/redisbank/
```

2. Run Redis Docker container locally

```
docker run -p 6379:6379 redislabs/redismod:latest
```

3. Execute the below CLI:

```
./mvnw clean package spring-boot:run
```

```
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ redisbank ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/ajeetraina/projects/redisbank/target/test-classes
[INFO] 
[INFO] <<< spring-boot-maven-plugin:2.4.5:run (default-cli) < test-compile @ redisbank <<<
[INFO] 
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.4.5:run (default-cli) @ redisbank ---
[INFO] Attaching agents: []

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.4.5)


```

4. Accessing the application

Navigate to http://localhost:8080 and login with user lars and password larsje

![My Image](https://github.com/redis-developer/redisbank/blob/main/redisbank1.png)
![My Image](https://github.com/redis-developer/redisbank/blob/main/redisbank2.png)

## Running on Azure Spring Cloud

1. Follow the steps from 'Running locally'
2. Make sure you are logged into the Azure CLI
3. Add the Azure Spring Cloud extension to the Azure CLI `az extension add --name spring-cloud` If you already have the extension, make sure it's up to date using `az extension update --name spring-cloud`
2. Create an Azure Spring Cloud instance using `az spring-cloud create -n acrebank -g rdsLroACRE -l northeurope` (this may take a few minutes)
3. Create an App in the newly created Azure Spring Cloud instance using `az spring-cloud app create -n acrebankapp -s acrebank -g rdsLroACRE --assign-endpoint true --runtime-version Java_11`
4. Modify the application.properties so it points to your newly created ACRE instance

```
spring.redis.host=your ACRE hostname
spring.redis.port=your ACRE port (default: 10000)
spring.redis.password= your ACRE access key
```

5. Modify the application.properties so the websocket config will point to the Azure Spring Cloud app instance endpoint createed in step 3.

```
stomp.host=your ASC app endpoint URL (Default: <appname>-<service-name>.azuremicroservices.io)
stomp.port=443
stomp.protocol=wss
```

6. Rebuild the app using `./mvnw package`
7. Deploy the app to Azure Spring Cloud using `az spring-cloud app deploy -n acrebankapp -s acrebank -g rdsLroAcre --jar-path target/redisbank-0.0.1-SNAPSHOT.jar`

## Troubleshooting tips on Azure Spring Cloud

To get the application logs:

`az spring-cloud app logs -n acrebankapp -g rdsLroAcre -s acrebank`

Note: project is compiled with JDK11 as that's currently the max LTS version that's supported by Azure Spring Cloud. Project will run fine when running locally or on other platforms up to JDK16.

## Known issues

1. Thread safety. Data is currently generated off of a single stream of transactions, which means it's the same for all users. Not a problem with the current iteration because it's single user, but beware when expanding this to multi-user.
2. Hardcoded values. Code uses hardcoded values throughout the code, these need to be replaced with proper variables.
