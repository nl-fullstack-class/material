# Spring Boot API 서버 구축

### Spring Initializr
* http://start.spring.io 접속
* Project `Gradle`
* Language `Java`
* Spring Boot `3.3.4`
* Project Metadata
    - Group `nl`
    - Artifact `springserver`
    - Name `springserver`
    - Packaging `jar`
    - Java `본인의 JDK버전에 맞는 걸로`
* Dependencies `Spring Web`, `Spring Data JPA`, `Lombok`, `MariaDB Driver`, `Validation`
* `Generate` -> zip 압축풀고 intellij로 실행


### MariaDB
1. spring boot 프로젝트의 `build.gradle` 안에 MariaDB Dependency가 잘 들어가 있는지 확인
```gradle
dependencies {
	runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
}
```
1. MariaDB 서버 실행 `$ mysql -u root -p`
1. > CREATE DATABASE \`fullstackdb`;
1. > USE \`fullstackdb`;
1. spring boot 프로젝트의 /src/main/resources/`application.properties` 파일에 아래와 같이 추가.
    ```properties
    spring.application.name=springserver

    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.show-sql=true
    spring.jpa.properties.hibernate.format_sql=true

    spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
    spring.datasource.url=jdbc:mariadb://localhost:3306/fullstackdb
    spring.datasource.username=root
    spring.datasource.password=0000
    ```
1. spring boot 프로젝트 application 실행해서 잘 되는지 확인


