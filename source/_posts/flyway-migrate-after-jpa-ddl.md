---
title: Flyway migrate晚于JPA建表语句
tags:
  - flyway
  - Spring Boot
id: '883'
categories:
  - - Java
comments: false
date: 2021-09-16 19:55:16
---

Spring Boot下通过`EntityManagerFactoryDependsOnPostProcessor`来确保Flyway的初始化执行晚于JPA。但是有时候我们会希望由JPA完成表结构的维护，Flyway用来修数据、基础数据的维护。这个时候如果flyway执行早于JPA的表结构维护，可能会导致表或字段不存在的异常。 所以我们重新实现`FlywayMigrationStrategy`的逻辑，在正常`migrate`时，不做具体事情，把flyway保存下来，在Spring完成初始化后`ContextRefreshedEvent`再执行`migrate`。

```Java
package org.xobo.configuration;

import org.flywaydb.core.Flyway;
import org.springframework.boot.autoconfigure.flyway.FlywayMigrationStrategy;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.context.event.EventListener;

@Configuration
public class MigrationConfiguration {

  public static class MyFlywayMigrationStrategy implements FlywayMigrationStrategy {
    private Flyway flyway;
    private boolean hasMigrated;

    public MyFlywayMigrationStrategy() {

    }

    @Override
    public void migrate(Flyway flyway) {
      this.flyway = flyway;
    }

    @EventListener
    public void handleContextRefresh(ContextRefreshedEvent event) {
      if (!hasMigrated && flyway != null) {
        flyway.migrate();
        hasMigrated = true;
      }
    }
  }

  @Bean
  FlywayMigrationStrategy FlywayMigrationStrategy() {
    return new MyFlywayMigrationStrategy();
  }
}
```