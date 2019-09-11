---
layout: post
title: JPA 와 mybatis 를 같은 Transaction 에서 사용하기
description: JPA 와 MyBatis 를 같은 Transaction 에서 사용하기
date:   2019-09-11 13:00:00 +0900
author: Jeongjin Kim
categories: JPA MyBatis Spring
tags:	JPA MyBatis Transaction SpringData
---

이 포스트는 같은 트랜잭션 내에 **JPA**와 **MyBatis**를 사용하는 방법을 **Spring Transaction** 을 이용한 예제를 보여준다.

# JpaTransactionManager
Spring 은 `PlatformTransactionManager` 인터페이스로 트랜잭션을 처리한다. 이 인터페이스의 구현체 중에 하나인 
`JpaTransactionManager` 는 단일 JPA `EntityManagerFactory` 를 유지하면서 스레드별로 `EntityManager`를 제공한다.

`JpaTransactionManager` 는 JPA 를 위해 주로 사용하지만 트랜젝션이 사용하고 있는 **DataSource**에 직접 접근이 가능하여 일반적인 JDBC 를 바로 사용할 수있다.

MyBatis의 서브 프로젝트인 **MyBatis-Spring**은 MyBatis의 트랜잭션 관리를 `SqlSession`이 아닌 **Spring Transaction**에 위임한다.

따라서 `JpaTransactionManager` 를 사용하면 **JPA**와 **MyBatis** 를 같은 트랜젝션으로 묶을 수 있게된다.

# 예제 코드
## 환경
* Spring-boot 2.1.7.RELEASE
* spring-boot-starter-data-jpa
* mybatis-spring-boot-starter
* h2
* lombok
* spring-boot-starter-test

### META-INF/persistence.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.2"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <!-- Define persistence unit -->
    <persistence-unit name="hello">
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.user_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="create"/>
        </properties>
    </persistence-unit>
</persistence>
```

### application.properties
```
spring.h2.console.enabled=true
```

### h2-mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="h2">
    <select id="selectUserInfo" resultType="cothe.UserInfo">
        SELECT id, name from userinfo
    </select>
</mapper>
```

### UserInfo.java
```java
import lombok.Getter;
import lombok.Setter;
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
@Setter
@Getter
public class UserInfo {
    @Id
    private Long id;
    private String name;
}
```

### Config.java
```java
import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

@Configuration
public class Config {
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/*.xml"));
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSession(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    @Bean
    public DataSource dataSource() {
        HikariDataSource hikariDataSource = new HikariDataSource();
        hikariDataSource.setDriverClassName("org.h2.Driver");
        hikariDataSource.setUsername("sa");
        hikariDataSource.setPassword("");
        hikariDataSource.setJdbcUrl("jdbc:h2:~/test");
        return hikariDataSource;
    }

    @Bean
    public EntityManagerFactory entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactoryBean.setDataSource(dataSource());
        entityManagerFactoryBean.setPersistenceUnitName("hello");
        entityManagerFactoryBean.setPersistenceXmlLocation("classpath:/META-INF/persistence.xml");
        entityManagerFactoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        entityManagerFactoryBean.afterPropertiesSet();
        return entityManagerFactoryBean.getObject();
    }

    @Bean
    public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

### JpaWithMybatisApplicationTests.java

```java
import org.apache.ibatis.session.SqlSession;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.orm.jpa.EntityManagerFactoryUtils;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.support.TransactionTemplate;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.transaction.TransactionManager;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
public class JpaWithMybatisApplicationTests {
    @Autowired
    private EntityManagerFactory emf;

    @Autowired
    private SqlSession sqlSession;

    @Autowired
    private JpaTransactionManager transactionManager;

    @Test
    public void contextLoads() {
        // transaction 단위
        TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
        transactionTemplate.execute( transactionStatus -> {
            EntityManager em = EntityManagerFactoryUtils.getTransactionalEntityManager(emf);
            UserInfo userInfo = new UserInfo();
            userInfo.setId(1L);
            userInfo.setName("새로운 이름");
            em.persist(userInfo);
            em.flush();

            List<Object> objects = sqlSession.selectList("h2.selectUserInfo", null);

            System.out.println("In transaction : " + objects.size());
            return userInfo;
        });

        List<Object> objects = sqlSession.selectList("h2.selectUserInfo", null);
        System.out.println("Out transaction : " + objects.size());

    }

    @After
    public void after() {
        emf.close();
    }
}
```
>JPA로 persist 한 뒤에 반드시 `EntityManager` 객체의 **`flush()`** 메소드를 호출해야한다. 그렇지 않으면 JPA 영속성 컨텍스트에만 저장되어 있고 실제 쿼리는 수행되지 않은 상태이기 때문에 MyBatis로는 아무것도 조회할 수 없다.

## 실행결과
```
In transaction : 2
Out transaction : 2
```