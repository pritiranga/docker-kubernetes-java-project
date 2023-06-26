# Use a base image with Java and Maven pre-installed
FROM maven:3.8.1-openjdk-11-slim

# Set the working directory in the container
WORKDIR /app

# Copy the project's POM and source files to the container
COPY pom.xml .
COPY src ./src

# Build the project
RUN mvn clean package

# Set the command to run the application
CMD ["java", "-jar", "target/your-application.jar"]
