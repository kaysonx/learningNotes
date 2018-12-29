FROM alpine:edge
LABEL maintainer="qspeng"
RUN apk add --no-cache openjdk8
VOLUME /tmp
COPY  logistics-service-*.jar logistics-service-*.jar
WORKDIR /app
EXPOSE 8082
ENTRYPOINT ["java", "-jar", "/logistics-service-*.jar"]