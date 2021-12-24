---
title: spring boot 多数据源
date: 2021-12-23 13:50:20
tags: java
categories: 语言
---

#### 1.配置文件

```
datasource:
    druid:
      football:
        url: jdbc:postgresql://xxx.xxx.xxx:5432/xxx
        username: xxx
        password: xxxxxx
        driver-class-name: org.postgresql.Driver
        filters: stat
        maxActive: 20
        initialSize: 1
        maxWait: 60000
        minIdle: 1
        timeBetweenEvictionRunsMillis: 60000
        minEvictableIdleTimeMillis: 300000
        validationQuery: select 'x'
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        poolPreparedStatements: true
        maxOpenPreparedStatements: 20
      basketball:
        url: jdbc:postgresql://xxx.xxx.xxx:5432/xxx
        username: xxx
        password: xxxxxx
        driver-class-name: org.postgresql.Driver
        filters: stat
        maxActive: 20
        initialSize: 1
        maxWait: 60000
        minIdle: 1
        timeBetweenEvictionRunsMillis: 60000
        minEvictableIdleTimeMillis: 300000
        validationQuery: select 'x'
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        poolPreparedStatements: true
        maxOpenPreparedStatements: 20
```

#### 2.DataSourceConfig配置类
```
package com.uecent.provider.api.module.common.config;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    @Bean(name = "footballDataSource")
    @ConfigurationProperties(prefix="spring.datasource.druid.football")
    public DataSource footballDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean(name = "basketballDataSource")
    @ConfigurationProperties(prefix="spring.datasource.druid.basketball")
    public DataSource basketballDataSource() {
        return DruidDataSourceBuilder.create().build();
    }
}
```

#### 3.针对两个数据源的配置类
MybatisBasketballConfig.java
```
package com.uecent.provider.api.module.common.config;

import com.baomidou.mybatisplus.core.MybatisConfiguration;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.type.JdbcType;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

/**
 * Mybatis配置
 */
@MapperScan(value = MybatisBasketballConfig.PACKAGE, nameGenerator = MybatisGenerator.class, sqlSessionTemplateRef = "basketballSqlSessionTemplate")
@Configuration
public class MybatisBasketballConfig {

    static final String PACKAGE = "com.uecent.provider.api.mapper.basketball";
    static final String MAPPER_LOCATION = "classpath:mapper/basketball/*.xml";

    @Bean(name = "basketballTransactionManager")
    public DataSourceTransactionManager basketballTransactionManager(@Qualifier("basketballDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "basketballSqlSessionFactory")
    public SqlSessionFactory basketballSqlSessionFactory(@Qualifier("basketballDataSource") DataSource dataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MybatisBasketballConfig.MAPPER_LOCATION));
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        configuration.setMapUnderscoreToCamelCase(true);
        configuration.setCacheEnabled(false);
        sessionFactory.setConfiguration(configuration);
        return sessionFactory.getObject();
    }


    @Bean(name = "basketballSqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplateOne(@Qualifier("basketballSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```

MybatisFootballConfig.java
```
package com.uecent.provider.api.module.common.config;

import com.baomidou.mybatisplus.core.MybatisConfiguration;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.type.JdbcType;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

/**
 * Mybatis配置
 */
@MapperScan(value = MybatisFootballConfig.PACKAGE, nameGenerator = MybatisGenerator.class, sqlSessionTemplateRef = "footballSqlSessionTemplate")
@Configuration
public class MybatisFootballConfig {

    static final String PACKAGE = "com.uecent.provider.api.mapper.football";
    static final String MAPPER_LOCATION = "classpath:mapper/football/*.xml";

    @Bean(name = "footballTransactionManager")
    @Primary
    public DataSourceTransactionManager footballTransactionManager(@Qualifier("footballDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "footballSqlSessionFactory")
    @Primary
    public SqlSessionFactory footballSqlSessionFactory(@Qualifier("footballDataSource") DataSource dataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MybatisFootballConfig.MAPPER_LOCATION));
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        configuration.setMapUnderscoreToCamelCase(true);
        configuration.setCacheEnabled(false);
        sessionFactory.setConfiguration(configuration);
        return sessionFactory.getObject();
    }

    @Bean(name = "footballSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate sqlSessionTemplateOne(@Qualifier("footballSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```