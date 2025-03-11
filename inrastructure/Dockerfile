FROM  maven:3-eclipse-temurin-21 as builder
WORKDIR application
COPY pom.xml /application
RUN mvn dependency:resolve
COPY . /application
RUN mvn clean package -DskipTests -X
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM openjdk:17.0.2-slim-buster
WORKDIR application
ENV APP_HOME /application
RUN groupadd -r appuser && useradd -r -g appuser appuser
#Create folder to save configuration files
RUN mkdir $APP_HOME/config
#Create folder with application logs
RUN mkdir $APP_HOME/logs
RUN chown -R appuser:appuser $APP_HOME
RUN chmod -R 777 $APP_HOME

#VOLUME $APP_HOME/logs
#VOLUME $APP_HOME/config

COPY --from=builder $APP_HOME/dependencies/ ./
COPY --from=builder $APP_HOME/snapshot-dependencies/ ./
COPY --from=builder $APP_HOME/spring-boot-loader/ ./
COPY --from=builder $APP_HOME/application/ ./

USER appuser
ENTRYPOINT ["java","-XX:+UseContainerSupport","-XX:MaxRAMFraction=1","-XshowSettings:vm","org.springframework.boot.loader.JarLauncher"]