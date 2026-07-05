FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /build
COPY pom.xml .
RUN apk add --no-cache maven && mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

RUN addgroup -g 10001 -S appgroup && \
    adduser -u 10000 -S appuser -G appgroup

COPY --from=builder /build/target/*.jar app.jar

RUN chown appuser:appgroup app.jar && chmod 700 app.jar

EXPOSE 8222

USER appuser:appgroup

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8222/actuator/health | grep UP || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]