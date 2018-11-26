---
layout: post
title: SpringBoot Gradle Multi Project 구성하고 nexus에 배포하기
description: SpringBoot Gradle Multi Project 구성하고 nexus에 배포하기
date:   2018-11-26 11:44:00 +0900
author: Jeongjin Kim
categories: Gradle Spring-boot
tags:	Gradle Spring-boot multi-project nexus
---

실행되는 프로젝트와 API 들을 묶어 놓은 프로젝트를 구분해야 하는 경우가 있다. 이때 이들을 별도의 프로젝트로 분리해놓으면 의존관계 때문에 꽤나 귀찮은 일들이 생길 수 있어서 하나의 큰 프로젝트안에 여러 소규모 프로젝트로 나누어 관리하기도 한다. 이렇게 나누어 관리하기 위한 설정을 기록해 놓고자 한다.

## 환경
* 스프링부트 2.1.0
* Gradle 4.10.2
* jdk 1.8

## 설정방법
### 1. root project

_settings.gradle_
```gradle
rootProject.name = 'root-project-name'

include 'sub-project-name-1'
include 'sub-project-name-2'
```
_build.gradle_

```gradle
buildscript {
    ext {
        springBootVersion = '2.1.0.RELEASE'
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath "io.spring.gradle:dependency-management-plugin:1.0.6.RELEASE"
    }
}

subprojects {
    group 'your.group'
    version "1.0.0-SNAPSHOT" // 프로젝트의 버전

    apply plugin: 'java'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

    sourceCompatibility = 1.8

    repositories {
        mavenCentral()
    }

    //전체 프로젝트에 공통으로 가지는 의존성을 추가 추가한다.
    dependencies {
        testCompile('org.springframework.boot:spring-boot-starter-test')
    }
}

project(':sub-project-name-1') {
    dependencies {
        compile project(':root-project-name')
    }
}
project(':sub-project-name-2') {
    dependencies {
        compile project(':root-project-name')
    }
}
```

### 2. sub projects for api
_build.gradle_

```gradle
bootJar { enabled = false } //단순히 api만 만들고 싶을 경우 추가해야 한다.
jar { enabled = true }

//sub-project 에서 필요한 의존성을 추가한다.
dependencies {
    compile('org.springframework.boot:spring-boot-starter')
}
```


## 추가 정리
### artifact 자동 업로드
내부 네트워크에 구성한 넥서스를 이용해서 리포지토리를 관리하고 있는데 자동으로 업로드 하기위한 스크립트를 정리해 둔다. 업로드 스크립트는 모든 프로젝트에 공통으로 적용되야해서 root 프로젝트에 추가하였다.

_build.gradle_

```gradle
//기존 subprojects 정의한 곳에 추가한다.
subprojects {
    apply plugin: 'maven'

    def nexusUrl = "http://nexus-url"
    def nexusUsername = 'id'
    def nexusPassword = 'password'

    // 기존에 정의한 repositories 에 추가한다.
    repositories {
        maven {
            url "${nexusUrl}/content/groups/public" //자체 개발한 프로젝트 jar를 import 할 수 있도록 nexus 주소 지정(물론 넥서스는 미리 구성되어야 한다)
        }
    }

    uploadArchives {
        repositories {
            mavenDeployer {
                repository(url: "${nexusUrl}/content/repositories/releases/") {
                    authentication(userName: "${nexusUsername}", password: "${nexusPassword}")
                }
                snapshotRepository(url: "${nexusUrl}/content/repositories/snapshots") {
                    authentication(userName: "${nexusUsername}", password: "${nexusPassword}")
                }
            }
        }
    }
}
```

### artifact에 소스와 자바독 추가하기
빌드할 때 소스와 자바독을 jar로 만들도록 정의하면 배포를 편리하게 할 수 있다. 모든 서브 프로젝트에 적용시키기위해 root project 에 적용을 했다.

_build.gradle_
```gradle
subprojects {
    task sourcesJar(type: Jar, dependsOn: classes) {
        classifier = 'sources'
        from sourceSets.main.allSource
    }

    task javadocJar(type: Jar, dependsOn: javadoc) {
        classifier = 'javadoc'
        from javadoc.destinationDir
    }
    javadoc {
        options.encoding = 'UTF-8'
    }
    artifacts {
        archives sourcesJar
        archives javadocJar
    }
}
```

## 마무리
이렇게 정리 해놓으면 간단한데 이상하게 gradle 문법은 헷갈리기도 하고 개념이 머리에 잘 안들와서 어려운 것 같다. 끝.
