# Multi-stage builds
- Introduce ability to run different parts of a build in multiple different environments, concurrently. 
- Multi-stage builds let you compile in one stage and copy the compiled binaries into a final runtime image. No need to bundle the entire compiler in your final image.

This time we gonna experiment with a Spring Boot project
- Specs: Java 21, Maven, Spring Web dependency
- `mvnw, pom.xml` for package env.
- the main stuff is in folder `main.java`
- File structure
```sh
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── spring_boot_docker
    │   │               └── SpringBootDockerApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
```
Add one controller at the main file
```java
@RestController
@SpringBootApplication
public class DemoApplication {

	@RequestMapping("/")
	public String home(){
		return "Hello world!";
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
Create `Dockerfile0`, we gonna renovate it later
```Dockerfile
FROM eclipse-temurin:21.0.2_13-jdk-jammy
WORKDIR /app

# dependencies
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline

# source
COPY src ./src

# run
EXPOSE 8080
CMD ["./mvnw", "spring-boot:run"]
```
- The base image is `eclipse-temurin:21.0.2_13-jdk-jammy`, containing JDK for Java, rmb this for later
- Resolve dependency via Maven binaries `mvnw`

Now build an image normally
```sh
# since the file name differ, specify it
$ docker build -t spring-helloworld -f Dockerfile0 .

# check the size
$ docker images | grep spring-hello
spring-helloworld   latest    47d8ea8f3455   2 hours ago    543MB
```

We can make the size smaller using build stage. Create another Dockerfile with name `Dockerfile1`

```Dockerfile
# Dockerfile1
# --------- build stage
FROM eclipse-temurin:21.0.2_13-jdk-jammy AS build
WORKDIR /app

# dependencies
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline

# source
COPY src ./src
RUN ./mvnw clean install

# --------- final stage
# notice different base img 
FROM eclipse-temurin:21.0.2_13-jre-jammy AS final
WORKDIR /app
# copy binaries
COPY --from=build /app/target/*.jar /app/*.jar
EXPOSE 8080
ENTRYPOINT [ "java", "-jar", "/app/*.jar" ]
```

- The first stage `build` will resolve all dependencies and compiled into binaries
- Second stage `final` only need to copy the binaries at `/app/target/` from stage `build`
- Notice the base image of `final` contains only runtime drivers, which is lighter in size.
- Read about `ENTRYPOINT` [here](https://spacelift.io/blog/docker-entrypoint-vs-cmd)

Such stages also result new partial name for the image
```sh
# build each stage
$ docker build -t spring-helloworld-build -f Dockerfile1 --target build .
$ docker build -t spring-helloworld-final -f Dockerfile1 --target final .

# or can be quicker
$ docker build -t spring-helloworld -f Dockerfile1 .

# look at the size brother
REPOSITORY                TAG       IMAGE ID       CREATED             SIZE
spring-helloworld-final   latest    4bc0f2ecfe29   About an hour ago   299MB
spring-helloworld-build   latest    dfed8910d933   About an hour ago   589MB
spring-helloworld         <none>    bac5dd9e430c   2 hours ago         543MB
```
- The reason for "quicker" command is the final stage `final` is the default target for building. 
- This means that if you don't explicitly specify a target stage using the `--target` flag in the docker build command, Docker will automatically build the last stage by default.





