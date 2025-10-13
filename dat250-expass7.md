### Task 7 

- Docker had already been used in the previous Tasks --> Installation had worked

- Therefore of course no error when executing the given command 

- Created a Dockerfile in my Project-Folder

- Created a .dockerignore for better build times 

- added for example git there 

- Created a docker-file with the build and the runtime stage 

```bash

# -------- 1) Build stage --------
FROM gradle:8-jdk17 AS builder
WORKDIR /home/gradle/project

# Copy project files
COPY . .

# Build the Spring Boot JAR (ohne Daemon)
RUN gradle clean bootJar --no-daemon

# -------- 2) Runtime stage --------
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app

# Create non-root user
RUN useradd -u 1000 appuser
USER appuser

# Copy built jar from previous stage
COPY --from=builder /home/gradle/project/build/libs/*.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]

```

- Needed to Change languageVersion = JavaLanguageVersion.of(17) in build.gradle

- Was able to start the app with docker afterwards
