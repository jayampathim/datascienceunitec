# Herd management API for Halter

Context:

  - [**Description**](#description)
  - [**Prerequisites**](#prerequisites)
  - [**Getting Started**](#getting-started)
  - [**Maven Dependencies**](#maven-dependencies)
  - [**Redis Configuration**](#redis-configuration)
  - [**Spring Service**](#spring-service)
  - [**Docker & Docker Compose**](#docker-docker-compose)
  - [**Build & Run Application**](#build-run-application)
  - [**Endpoints with Swagger**](#endpoints-with-swagger)
  - [**Demo**](#demo)


## [Description]

● This API facilitates to mange CRUD operations of cows.<br/>
● API has been implementd by using  Java SpringBoot REST service.<br/>
● Potgres is used as the persistence layer while Redis is used as a cache layer.<br/>

## Prerequisites

To run Spring Boot based Account REST API it is prerequisite to have Java 8 & Docker installed on your machine.<br/>

● [Steps to Install Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)<br/>
● [Steps to Install Docker](https://docs.getting-starteddocker.com/install/) <br/>
● Mavenis used to build dependancy of the project.
● It's not required to install PostgreSQL or Redis as services are run on Dcoker containers.<br/>
● But important to have undersandig of how PostgreSQL & Redis works (https://www.postgresql.org & https://redis.io )<br/>


## Getting Started

```xml
The application configure PostgreSQL and Redis with Java Spring Boot.Redis is most popular tool to use for caching. It is widely usage in the web application development.the application uses Dockerfile and docker-compose.yml files to dockerize containers.Also Redis has been added with cache tag to docker-compose.yml file.Docker compose is a very useful tool to run more than one containers.<br/>

* PostgreSQL Database.
* Redis as
* API Endpoints for CRUD operations.
* Dockerfile

## Maven Dependencies
```xml
Below dependencies are required to build and run the project.All dependencies are placed in the pom.xml.

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
</dependency>	
<dependency>
	<groupId>org.postgresql</groupId>
	<artifactId>postgresql</artifactId>
	<scope>runtime</scope>
</dependency>
	<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.9.2</version>
</dependency>

## Redis Configuration

```java
@Configuration
@AutoConfigureAfter(RedisAutoConfiguration.class)
@EnableCaching
public class RedisConfig {

	@Autowired
	private CacheManager cacheManager;

	@Value("${spring.redis.host}")
	private String redisHost;

	@Value("${spring.redis.port}")
	private int redisPort;

	@Bean
	public RedisTemplate<String, Serializable> redisCacheTemplate(LettuceConnectionFactory redisConnectionFactory) {
		RedisTemplate<String, Serializable> template = new RedisTemplate<>();
		template.setKeySerializer(new StringRedisSerializer());
		template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	public CacheManager cacheManager(RedisConnectionFactory factory) {
		RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
		RedisCacheConfiguration redisCacheConfiguration = config
				.serializeKeysWith(
						RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
				.serializeValuesWith(RedisSerializationContext.SerializationPair
						.fromSerializer(new GenericJackson2JsonRedisSerializer()));
		RedisCacheManager redisCacheManager = RedisCacheManager.builder(factory).cacheDefaults(redisCacheConfiguration)
				.build();
		return redisCacheManager;
	}

}
```

Dockerfile:
```xml
FROM openjdk:11
ADD ./target/herd-management-service-0.0.1-SNAPSHOT.jar /usr/src/herd-management-service-0.0.1-SNAPSHOT.jar
WORKDIR usr/src
ENTRYPOINT ["java","-jar", "herd-management-service-0.0.1-SNAPSHOT.jar"]
```

Docker compose file: docker-compose.yml

```yml
version: '3'

services:
  db:
    image: "postgres"
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ekoloji
  cache:
    image: "redis"
    ports: 
      - "6379:6379"
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_DISABLE_COMMANDS=FLUSHDB,FLUSHALL
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db/postgres
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: ekoloji
      SPRING_REDIS_HOST: cache
      SPRING_REDIS_PORT: 6379
    depends_on:
      - db
      - cache

## Build & Run Application

 Please follow the below steps to build and run the application.Docker Compose Build and Run<br/>
  ● Go to the project directory and open the terminal from outside of the project directory.<br/>
  ● Run the command 'mvn clean install -DskipTests' <br>
  ● Run the command  'docker-compose build --no-cache'- Docker Compose Build <br/>
  ● Run the command  'docker-compose up --force-recreate'- Docker Compose Run <br/>

After running the application you can visit `http://localhost:8080/swagger-ui.html`<br/>	

## Endpoints with Swagger

You can see the endpoint in `http://localhost:8080/swagger-ui.html` page.Application used Swagger for visualization of API endpoints.<br>

## Screenshots of Demo

<div align="center">
  API endpoint<br\>
  	<img src="https://github.com/jayampathim/datascienceunitec/blob/master/endpoints_interface.png" alt="Spring Boot + Redis + PostgreSQL Caching">
  Create a new cow<br\>
  	<img src="https://github.com/jayampathim/datascienceunitec/blob/master/endpoints_interface.png" alt="Spring Boot + Redis + PostgreSQL Caching">
  Get all cows <br\>
  	<img src="https://github.com/jayampathim/datascienceunitec/blob/master/getAllCows.png" alt="Spring Boot + Redis + PostgreSQL Caching">
  Update a cow <br\>
        <img src="https://github.com/jayampathim/datascienceunitec/blob/master/update_cow.png" alt="Spring Boot + Redis + PostgreSQL Caching">	
</div>

