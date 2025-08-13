FROM tomcat:9.0.60-jre11-openjdk-slim-buster

RUN echo 'deb http://archive.debian.org/debian buster main contrib non-free\n\
deb http://archive.debian.org/debian buster-updates main contrib non-free\n\
deb http://archive.debian.org/debian-security buster/updates main contrib non-free' > /etc/apt/sources.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99-no-check-valid-until && \
    apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

ADD spring-form.war /usr/local/tomcat/webapps/

EXPOSE 8888

CMD ["catalina.sh", "run"]
