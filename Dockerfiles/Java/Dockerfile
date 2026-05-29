# Use a single image that contains both Maven and JDK
FROM maven:3.9.2-openjdk-17

WORKDIR /app

# Copy the entire project into the container
COPY . .

# Build the application and package the JAR
RUN mvn package -DskipTests

# Expose the application port
EXPOSE 8080

# Run the application directly from the target folder
CMD ["java", "-jar", "target/myapp.jar"]
