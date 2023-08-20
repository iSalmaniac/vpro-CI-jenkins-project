FROM openjdk:11 AS BUILD_IMAGE
RUN apt update && apt install maven -y
RUN git clone https://github.com/iSalmaniac/vpro-CI-jenkins-project.git
RUN cd vpro-CI-jenkins-project && git checkout docker && mvn install
FROM tomcat:9-jre11 
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=BUILD_IMAGE vpro-CI-jenkins-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8081
CMD ["catalina.sh", "run"]
