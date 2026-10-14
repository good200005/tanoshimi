# syntax=docker/dockerfile:1
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app
COPY gradlew settings.gradle build.gradle ./
COPY gradle gradle
# Windows 에서 체크아웃된 gradlew 의 CRLF 제거
RUN sed -i 's/\r$//' gradlew && chmod +x gradlew
# 의존성 레이어 캐시(소스가 바뀌어도 재다운로드 방지)
RUN ./gradlew --no-daemon dependencies > /dev/null 2>&1 || true
COPY src src
RUN ./gradlew --no-daemon bootJar -x test && rm -f build/libs/*-plain.jar

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
RUN mkdir -p /tmp/uploads
# Render 무료 플랜(512MB) 기준 메모리 설정. PORT 는 Render 가 주입한다.
ENV JAVA_OPTS="-XX:MaxRAMPercentage=70 -XX:+UseSerialGC -Xss512k"
EXPOSE 8080
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -Dserver.port=${PORT:-8080} -jar /app/app.jar"]
