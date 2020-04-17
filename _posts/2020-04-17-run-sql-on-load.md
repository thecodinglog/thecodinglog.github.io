---
layout: post
title: ApplicationRunner-Spring Application 시작할 때 Query 실행하기
description: Spring Application 시작할 때 Query 실행하기
date:   2020-04-17 09:00:00 +0900
author: Jeongjin Kim
categories: spring boot
tags:	execute sql query
---

이 포스트는 SprinBboot Application이 구동될 때 sql 파일을 직접 읽어서 실행하는 샘플을 보여드립니다.

Springboot 는 모든 빈이 초기화되고 애플리케이션이 실행되기 전에 `ApplicationRunner` interface를 찾아서 실행시켜줍니다.
이 타이밍에 sql을 실행하는 구문을 넣어서 돌려봅니다.

아래 샘플은 샘플일 뿐 프로덕트에 그대로 사용하지는 마시기 바랍니다.

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


```java
@Component
@Slf4j
public class Runner implements ApplicationRunner {
    final
    JdbcTemplate jdbcTemplate;
    final
    EntityManagerFactory entityManagerFactory;

    public Runner(JdbcTemplate jdbcTemplate, EntityManagerFactory entityManagerFactory) {
        this.jdbcTemplate = jdbcTemplate;
        this.entityManagerFactory = entityManagerFactory;
    }

    @Override
    public void run(ApplicationArguments args) throws FileNotFoundException {
        log.debug("Runner start!");
        URL resource = getClass().getClassLoader().getResource("data-mysql.sql");
        if (resource == null)
            return;
        StringBuilder stringBuilder = new StringBuilder();
        try (Stream<String> lines = Files.lines(Paths.get(resource.toURI()), StandardCharsets.UTF_8)) {
            lines.forEach(stringBuilder::append);
        } catch (IOException | URISyntaxException e) {
            e.printStackTrace();
        }

        String[] split = stringBuilder.toString().split(";");

        JpaTransactionManager transactionManager = new JpaTransactionManager(entityManagerFactory);
        TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
        transactionTemplate.execute(transactionStatus -> {
            for (String s : split) {
                if (s.startsWith("#"))
                    continue;
                jdbcTemplate.execute(s);
            }
            return null;

        });
    }
}
```
