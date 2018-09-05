---
layout: post
title: Cloud Native Java 따라하기 2
description: Cloud Native Java 클라우드 네이티브 자바를 따라해보자 spring-boot 스프링부트
date:   2018-09-05 13:00:00 +0900
author: Jeongjin Kim
categories: Cloud Spring Java
tags:	Cloud Spring Java Spring-boot
---
앞선 Cloud Native Java 포스트에서는 스프링 프레임워크에서 어떻게 설정 값을들 객체와 매핑하여 사용할 수 있는지 설정했다. 스프링부트에서는 어떻게 쓸 수 있는지 알아보자.

## 스프링 부트 방식의  설정
#### 설정정보의 우선순위
1. 명령행 인자
2. `java:comp/env` 에 있는 JNDI 속성
3. `System.getProperties()` 로 읽어오는 속성
4. 운영체제의 환경 변수
5. jar 외부에 존재하는 `application.properties` 파일이나 `application.yml` 파일에 정의된 속성
6. jar 내부에 존재하는 `application.properties` 파일이나 `application.yml` 파일에 정의된 속성
7. `@Configuration` 이 붙은 클래스에 `@PropertySource` 로 지정된 곳에 있는 속성
8. `SpringApplication.getDefaultProperties()` 로 읽어올 수 있는 기본값

특정 프로파일이 활성화되어 있으면 프로파일 이름을 기준으로 `src/main/resources/application-활성프로파일이름.properties` 파일에 있는 속성 정보를 자동으로 일어올 수 있다.
클래스 패스에 SnakeYAML 라이브러리가 있으면 자동으로 YAML 파일에서도 properties 파일과 같은 방식으로 속성정보를 가져올 수 있다.

`$CONFIGURATION_PROJECT` 라는 환경변수를 사용하거나 `-Dconfiguration.projectname` 옵션을 추가해서 스프링부트 애플리케이션을 실행하면 소스코드에서 `configuration.projectName` 이라는 키로 설정값을 읽어올 수 있다.

프로젝트에서 스프링부트를 사용할 수 있도록 pom.xml 을 변경하자. parent에 `spring-boot-starter-parent` 를 선언했다. 이걸 이용하면 하위 의존성 추가할때 버전을 귀찮게 맞춰야 하는 일을 덜 수 있다. 추가적으로 편의를 위해 lombok 을 의존성에 넣었다.

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cothe</groupId>
    <artifactId>springConfigurer</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```
`SpringbootConfigApplication.java`

```java
package cothe;

@EnableConfigurationProperties // 스프링에게 설정정보를 @ConfigurationProperties 가 붙은 POJO 에 매핑하라고 알려줌
@SpringBootApplication
@Slf4j
public class SpringbootConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootConfigApplication.class);
    }

    @Autowired
    public SpringbootConfigApplication(ConfigurationProjectProperties cp){
        log.info("configurationProjectProperties.projectName = {}", cp.getProjectName());
    }
}

@Component // configuration 으로 시작하는 설정값은 @ConfigurationProperties("configuration") 이 붙은 빈에 매핑됨
@ConfigurationProperties("configuration")
@Getter
@Setter
class ConfigurationProjectProperties{
    private String projectName; // configuration.projectName 으로 지정된 설정값이 할당 됨
}
```

`src/main/resources/` 에 `application.yml` 파일을 추가해서 아래 내용을 넣는다.

```yml
configuration:
  projectName: Spring Boot Project
```

애플리케이션을 실행하면 새로 추가한 설정파일 정보가 POJO 객체에 매핑된 것을 확인 할 수 있다.

![](/assets/2018-09-05-cloud-native-java-2/2018-09-05-cloud-native-java-2_093225.png)

요약해보면 `@EnableConfigurationProperties` 애노테이션을 붙여서 스프링에게 설정정보를 매핑할 수 있도록 준비 시킨다. 매핑될 클래스는 `@ConfigurationProperties` 애노테이션으로 지정하는데 이에 대한 값으로 prefix 값을 줄 수 있다. 이 prefix와 일치하는 설정 중 POJO 클래스에 매핑 가능한 필드를 찾아서 매핑한다.