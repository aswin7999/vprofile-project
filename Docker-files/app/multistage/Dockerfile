FROM maven:3.9-eclipse-temurin-11 AS build_image
RUN git clone https://github.com/<aswin7999>/vprofile-project.git
RUN cd vprofile-project && git checkout docker && mvn install

FROM tomcat:9-jre11
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=build_image /vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]