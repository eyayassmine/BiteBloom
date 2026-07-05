# Build stage
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /build
# Copy pom.xml first (Docker caches this layer — faster rebuilds)
COPY pom.xml .
# Download dependencies separately (cached unless pom.xml changes)
RUN apk add --no-cache maven && mvn dependency:go-offline -B
# Now copy source code
COPY src ./src
# Build the jar, skip tests (already ran them separately)
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Security: create non-root user and group
# UIDs above 10000 avoid overlap with system users
RUN addgroup -g 10001 -S appgroup && \
    adduser -u 10000 -S appuser -G appgroup

# Copy only the jar from build stage (not source code!)
COPY --from=builder /build/target/*.jar app.jar

# Give ownership to non-root user
RUN chown appuser:appgroup app.jar && chmod 700 app.jar

# Document which port this service uses
EXPOSE 8761

# Switch to non-root user — runs everything after this as appuser
USER appuser:appgroup

# Health check using actuator endpoint
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8761/actuator/health | grep UP || exit 1

# Run the jar
ENTRYPOINT ["java", "-jar", "app.jar"]

