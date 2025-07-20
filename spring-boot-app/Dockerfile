# You can change this base image to anything else
# But make sure to use the correct version of Java
FROM adoptopenjdk/openjdk11:alpine-jre

# Simply the artifact path
ARG artifact=target/spring-boot-demo-1.0.0.jar

WORKDIR /opt/app

COPY target/*.jar app.jar

# This should not be changed
ENTRYPOINT ["java","-jar","app.jar"]
