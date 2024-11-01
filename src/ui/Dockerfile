# Build Stage
FROM amazonlinux:2 as build-env

# Install dependencies
RUN yum install -y \
    maven \
    java-17-amazon-corretto-headless \
    which && \
    yum clean all

VOLUME /tmp
WORKDIR /

COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

RUN ./mvnw dependency:go-offline -B -q

COPY ./src ./src

RUN ./mvnw -DskipTests package -q && \
    mv /target/ui-0.0.1-SNAPSHOT.jar /app.jar

# Package Stage
FROM amazonlinux:2

# Install runtime dependencies
RUN yum install -y \
    java-17-amazon-corretto-headless \
    shadow-utils && \
    yum clean all

ARG aws_opentelemetry_agent_version=1.24.0

ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

# Add a non-root user for application execution
RUN useradd \
    --home "/app" \
    --create-home \
    --user-group \
    --uid "$APPUID" \
    "$APPUSER"

# Download OpenTelemetry agent
RUN curl -L https://github.com/aws-observability/aws-otel-java-instrumentation/releases/download/v${aws_opentelemetry_agent_version}/aws-opentelemetry-agent.jar -o /opt/aws-opentelemetry-agent.jar && \
    curl -L https://raw.githubusercontent.com/aws-observability/aws-otel-java-instrumentation/v${aws_opentelemetry_agent_version}/licenses/licenses.md -o aws-opentelemetry-agent-licenses.md

ENV JAVA_TOOL_OPTIONS=-javaagent:/opt/aws-opentelemetry-agent.jar
ENV OTEL_JAVAAGENT_ENABLED=false

WORKDIR /app
USER appuser

# Copy necessary files and the built application from the build stage
COPY ./ATTRIBUTION.md ./LICENSES.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]

