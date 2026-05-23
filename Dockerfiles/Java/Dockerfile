# Stage 1: Build Stage
FROM maven:3.9.2-openjdk-17 AS builder

WORKDIR /app

# Copy Maven configuration
COPY pom.xml .

# Download dependencies separately for caching
RUN mvn dependency:go-offline

# Copy source code
COPY src ./src

# Build application
RUN mvn package -DskipTests


# Stage 2: Runtime Stage
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copy generated jar from builder stage
COPY --from=builder /app/target/myapp.jar myapp.jar

# Expose application port
EXPOSE 8080

# Run application
CMD ["java", "-jar", "myapp.jar"]