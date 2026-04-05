



# Stage 1: Build
FROM maven:3.9.2-openjdk-17 AS builder
WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build jar
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Runtime
FROM openjdk:17-jdk-slim
WORKDIR /app

# Copy built jar from builder
COPY --from=builder /app/target/myapp.jar ./myapp.jar

EXPOSE 8080
CMD ["java", "-jar", "myapp.jar"]