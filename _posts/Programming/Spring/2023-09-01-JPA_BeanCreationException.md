---
title: "[JPA] BeanCreationException Error"

categories:
  - spring
tags:
  - [spring, jpa]

toc: true
toc_sticky: true

date: 2023-09-01
last_modified_at: 2023-09-01
---


# Error 

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: [PersistenceUnit: default] Unable to build Hibernate SessionFactory; nested exception is org.hibernate.exception.SQLGrammarException: Unable to build DatabaseInformation [Column "start_value" not found [42122-214]] [n/a]

```

# 해결방안

>   application.property 파일의 ddl 설정을 아래와 같이 변경

    #spring.jpa.hibernate.ddl-auto=update

    spring.jpa.hibernate.hbm2ddl.auto=update

# etc


[Spring Boot Reference Documentation
](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data.sql.jpa-and-spring-data.creating-and-dropping)


스프링 공식 문서에 나와있듯이 

> spring.jpa.hibernate.ddl-auto=update 

> spring.jpa.hibernate.hbm2ddl.auto=update

둘 다 동일한 설정을 하는 코드이다. ddl-auto 는 spring.jpa.hibernate.hbm2ddl.auto 설정을 위한 shortcut 이라고 설명이 나와있다. 하지만 어떤 이유에선지 ddl-auto 는 에러가 발생한다. 

H2 데이터베이스를 사용 중인데 1.4 버전을 사용하고 있기에 발생하는 문제같기도 하다. 