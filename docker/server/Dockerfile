#Multi-Stage build

#Build application stage
#We need maven.
FROM maven:3.5.3-jdk-9
ARG JAVA_AGENT_VERSION=1.19.0
ARG JAVA_AGENT_BRANCH=1.x
ARG JAVA_AGENT_REPO=elastic/apm-agent-java

WORKDIR /usr/src/java-code
#build the application
RUN git clone https://github.com/synapticielRD/spring-petclinic.git
WORKDIR /usr/src/java-code/spring-petclinic
RUN mvn -q -B package -DskipTests

RUN mkdir /usr/src/java-app
RUN cp -v /usr/src/java-code/spring-petclinic/target/*.jar /usr/src/java-app/app.jar


## Download the APM java gent
## https://repo1.maven.org/maven2/co/elastic/apm/elastic-apm-agent/${JAVA_AGENT_VERSION}/elastic-apm-agent-${JAVA_AGENT_VERSION}.jar
RUN curl -L https://repo1.maven.org/maven2/co/elastic/apm/elastic-apm-agent/${JAVA_AGENT_VERSION}/elastic-apm-agent-${JAVA_AGENT_VERSION}.jar

FROM openjdk:9

RUN export
WORKDIR /app

COPY --from=0 /usr/src/java-app/*.jar ./

CMD java -javaagent:/app/elastic-apm-agent-${JAVA_AGENT_VERSION}.jar\
                                        -Dspring.profiles.active=${JAVA_PROFILE:-hsqldb,spring-data-jpa}\
                                        -Dserver.port=${SERVER_PORT:-}\
                                        -Delastic.apm.application_packages=org.springframework.samples.petclinic\
                                        -Dserver.context-path=/petclinic/\
                                        -Dspring.messages.basename=messages/messages\
                                        -Dlogging.level.org.springframework=${LOG_LEVEL:-INFO}\
                                        -Dsecurity.ignored=${SECURITY_IGNORED:-/**}\
                                        -Dspring.datasource.initialize=${INITIALIZE_DB:-false}\
                                        -Dbasic.authentication.enabled=${AUTHENTICATION_ENABLED:-false}\
                                        -Dserver.address=${SERVER_ADDRESS:-0.0.0.0}\
                                        -Dspring.datasource.url=${DATABASE_URL:-jdbc:hsqldb:mem:petclinic}\
                                        -Dspring.datasource.username=${DATABASE_USERNAME:-sa}\
                                        -Dspring.datasource.password=${DATABASE_PASSWORD:-}\
                                        -Dspring.datasource.driver-class-name=${DATABASE_DRIVER:-}\
                                        -Dspring.jpa.database=${DATABASE_DIALECT:-HSQL}\
                                        -Dspring.jpa.database-platform=${DATABASE_PLATFORM:-org.hibernate.dialect.HSQLDialect}\
                                        -Dspring.jpa.hibernate.ddl-auto=${DDL_AUTO:-none}\
                                        -Dspring.datasource.schema=${DATASOURCE_SCHEMA:-classpath*:db/hsqldb/initDB.sql}\
                                        -Dspring.datasource.data=${DATASOURCE_DATA:-classpath*:db/hsqldb/populateDB.sql}\
                                        -Delastic.apm.service_name=${ELASTIC_APM_SERVICE_NAME:-spring-petclinic}\
                                        -Delastic.apm.service_version=${ELASTIC_APM_SERVICE_VERSION:-1.0.0}\
                                        -Delastic.apm.span_frames_min_duration=${ELASTIC_APM_SPAN_FRAMES_MIN_DURATION:-5ms}\
                                        -Delastic.apm.capture_body=${ELASTIC_APM_CAPTURE_BODY:-off}\
                                        -Delastic.apm.environment=production\
                                        -Delastic.apm.transaction_sample_rate=${APM_SAMPLE_RATE:-1.0}\
                                        -Delastic.apm.server_urls=${ELASTIC_APM_SERVER_URL:-http://localhost:8200}\
                                        -Delastic.apm.verify_server_cert=false\
                                        -Delastic.apm.ignore_urls=/health,/metrics*,/jolokia\
                                        -Delastic.apm.log_file=/var/log/apps/apm-spring-petclinic\
                                        -Delastic.apm.enable_log_correlation=true\
                                        -jar /app/app.jar
