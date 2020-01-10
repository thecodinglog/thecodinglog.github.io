---
layout: post
title: 스프링 빈 동적 생성
description: Registering bean dynamically using custom configurations
date:   2019-01-29 11:44:00 +0900
author: Jeongjin Kim
categories: spring
tags: ContextRefreshedEvent Spring BeanPostProcessor BeanFactoryPostProcessor BeanDefinitionRegistryPostProcessor mybatis
---

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>


프로젝트에 커스텀 설정정보를 이용해서 **동적으로 스프링 빈 등록**을 하려고 한다. 이 기능이 필요했던 이유는 _mybatis_ 때문인데 mybatis를 사용하려면 `SqlSessionTemplate`, `SqlSessionFactory`, `Datasource`, `TransactionManager` 등 접속 정보 하나를 추가할 때 마다 하려면 다른 Class들도 빈으로 같이 등록해줘야 했기 때문이다. DataSource 3개만 돼도 비슷비슷한 이름으로 많은 양의 코드가 증가하게 되는데 이는 필시 개발자가 오타 등으로 예상치 못한 **오류를 유발**할 수 있다.

그래서 최소한의 정보(데이터소스 정보)만 가지고 mybatis를 사용할 수 있도록 구현하고자 한다.

## 빈 후커

스프링의 빈 정보를 수집해서 서로의 의존관계를 조사하고 인스턴트화하는 과정을 후킹할 수 있는 `BeanDefinitionRegistryPostProcessor`, `BeanFactoryPostProcessor`, `BeanPostProcessor` Interface가 있다.
세 Interface는 수행되는 시점과 역할이 다르다.
호출되는 순서는 나열한 순서대로 인데 `BeanDefinitionRegistryPostProcessor` 는 빈 정의를 **등록**하는데 촛점이 맞춰진 Inteface이다. `BeanFactoryPostProcessor` 는 빈 정의 자체를 **재정의**하거나 프로퍼티를 추가하기 위해서 주로 사용한다. 이 두 인터페이스 모두 **인스턴스화가 되기 전에 호출**된다. 반면 `BeanPostProcessor`는 **인스턴스화된 빈을 변경**(예를 들면 마커 인터페이스가 있는지 확인해본다던지, 인스턴스를 Proxy로 감싼다던지)하기 위해 주로 사용한다. 따라서 모든 빈마다 호출된다.

## 후킹 인터페이스의 한계점

처음에는 이런 Interface이용해서 기능을 구현하려고 했으나 몇 가지 불편한 점이 있었다.
SpringBoot는 `Application.yml` 에 설정한 정보를 그대로 객체로 바꿔주는 기능을 제공하는데 이 객체에 접근해서 정보를 가져오는게 너무나도 편리하기 때문에(파일을 그대로 읽어서 파싱한 다음 사용하는 것 보다) 어떻게든 이것을 사용하려고 했다.
그러다 보니 스프링에서 제공하는 hooking interface에서 구현은 문제가 있었다. **설정 정보 객체가 만들어지기 전에 interface가 호출되기 때문이다.**

## 해결책 탐색

등록된 모든 빈이 초기화 되고 난 뒤 동적으로 빈 객체를 넣어주는 방법이 있지 않을까 해서 조사하였다.
Spring Context 초기화 됐을 때 내가 구현한 코드를 실행하게 하려면 어떻게 해야할까?
Spring은 Spring Context에 어떤 변화가 생기면 Event를 발생하는데 그중에 하나가 `ContextRefreshedEvent`이다. 모든 빈이 다 등록되고 난 뒤 발생하는 이벤트인데 이 이벤트 핸들러를 구현하여 문제를 해결해 봤다.
Spring Context를 이용해서 새로운 빈을 등록할 때 주의해야 할 것은 **registerBeanDefinition** 메소드는 의존성 정보를 이용해서 자동으로 주입되고 하는 DI 가 동작하지 않는다. 따라서 커스텀으로 빈을 만들때 필요한 의존성은 직접 넣어줘야 한다.

## 구현
### VO
```java
@Component
@ConfigurationProperties(prefix = "repository")
public class RepositoryConfigVO {
    @Setter
    @Getter
    private List<Repository> repositories;

    @Setter
    @Getter
    private String mapperLocations;
    @Setter
    @Getter
    private String configLocation;

    public static class Repository {
        @Setter
        @Getter
        private String name;
        @Setter
        @Getter
        private Map<String, String> dataSource;
    }
}
```
### ContextRefreshedEvent 핸들러
```java 
@Component
@Slf4j
public class RepositoryConfigInitializerEventListener {
    @EventListener
    public void onApplicationEvent(ContextRefreshedEvent event) {
        ApplicationContext applicationContext = event.getApplicationContext();

        RepositoryConfigVO repositoryConfigVO = null;
        try {
            repositoryConfigVO = applicationContext.getBean("repositoryConfigVO", RepositoryConfigVO.class);
        } catch (BeansException e) {
            log.warn("RepositoryConfig 설정이 없습니다.");
            return;
        }

        if(repositoryConfigVO.getRepositories() == null){
            log.warn("Repository 설정이 없습니다.");
            return;
        }

        BeanDefinitionRegistry beanFactory = (BeanDefinitionRegistry) ((GenericApplicationContext) applicationContext).getBeanFactory();

        String mapperLocations = repositoryConfigVO.getMapperLocations();
        String configLocation = repositoryConfigVO.getConfigLocation();

        if(mapperLocations == null) mapperLocations = "classpath:/mappers/**/*.xml";

        if(configLocation == null) configLocation = "classpath:/mybatis-config.xml";
        
        for (RepositoryConfigVO.Repository repository : repositoryConfigVO.getRepositories()) {
            String repositoryName = repository.getName();

            // DataSource 등록
            GenericBeanDefinition dataSourceBeanDefinition = new GenericBeanDefinition();
            dataSourceBeanDefinition.setBeanClass(HikariDataSource.class);
            dataSourceBeanDefinition.setPropertyValues(new MutablePropertyValues(repository.getDataSource()));

            beanFactory.registerBeanDefinition(repositoryName + "DataSource", dataSourceBeanDefinition);

            // SqlSessionFactory 등록
            AbstractBeanDefinition sqlSessionFactoryBeanDefinition = BeanDefinitionBuilder.genericBeanDefinition(SqlSessionFactoryBean.class)
                    .addPropertyReference("dataSource", repositoryName + "DataSource")
                    .addPropertyValue("mapperLocations", mapperLocations)
                    .addPropertyValue("configLocation", configLocation)
                    .getBeanDefinition();

            beanFactory.registerBeanDefinition(repositoryName + "SqlSessionFactory", sqlSessionFactoryBeanDefinition);
            
            // SqlSessionTemplate 등록
            AbstractBeanDefinition sqlSessionTemplateBeanDefinition = BeanDefinitionBuilder.genericBeanDefinition(SqlSessionTemplate.class)
                    .addConstructorArgReference(repositoryName + "SqlSessionFactory")
                    .getBeanDefinition();

            beanFactory.registerBeanDefinition(repositoryName + "SqlSessionTemplate", sqlSessionTemplateBeanDefinition);
            
            // Dao 등록
            AbstractBeanDefinition commonDaoBeanDefinition = BeanDefinitionBuilder.genericBeanDefinition(CommonDaoImpl.class)
                    .addConstructorArgValue(repositoryName + "DataSourceTransactionManager")
                    .addConstructorArgReference(repositoryName + "SqlSessionTemplate")
                    .getBeanDefinition();

            beanFactory.registerBeanDefinition(repositoryName + "Dao", commonDaoBeanDefinition);
            
            // TransactionManager 등록
            AbstractBeanDefinition transactionManagerBeanDefinition = BeanDefinitionBuilder.genericBeanDefinition(DataSourceTransactionManager.class)
                    .addPropertyReference("dataSource", repositoryName + "DataSource")
                    .getBeanDefinition();

            beanFactory.registerBeanDefinition(repositoryName + "DataSourceTransactionManager", transactionManagerBeanDefinition);
        }
    }
}
```

## 마무리

실행 환경 변수를 가지고 위와 같이 동적 빈생성 코드를 작성할 수도 있다. 그 때에는 `BeanDefinitionRegistryPostProcessor` 인터페이스를 구현하는 것이 훨씬 편리할 수 있다.
`ContextRefreshedEvent` 이벤트는 초기화되거나 갱신됐을 때 발생하는 이벤트이다. 여기서 이 갱신이라는 의미가 정확하게 어떤 의미인지 아직까지는 와닫지 않고 언제 갱신되는지는 조금더 확인을 해봐야 할 것같다.
