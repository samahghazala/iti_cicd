# FROM openjdk:11 AS build_image
FROM maven:3.9.6-eclipse-temurin-17 AS build_image 
RUN apt update && apt install git -y
RUN git clone https://github.com/devopshydclub/vprofile-project.git
WORKDIR /vprofile-project

RUN git checkout docker && \
    # Inject a visible banner at the top of the webpage body
    ARG BUILD_NUMBER=1
    find src/main/webapp/ -name "*.jsp" -exec sed -i "s/<body>/<body><div style=\"background-color: #007bff; color: white; text-align: center; padding: 15px; font-size: 22px; font-weight: bold;\">🚀 BUILD #${BUILD_NUMBER} LIVE<\/div>/g" {} + && \
    mvn install


FROM tomcat:9-jre11
LABEL "Project"="Vprofile"
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=build_image vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
