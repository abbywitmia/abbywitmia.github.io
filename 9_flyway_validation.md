# 跬步9 Flyway validation

## Situation
出于安全性的考虑,我们在生产环境禁止了程序数据库账号的DDL权限.
这就导致flyway的migration不可能成功.
所以我们在生产环境禁用了flyway防止migration失败阻止应用启动.
但是这样我们就无法使用flyway了.
不用flyway我们依然存在不同环境schema一致性的问题,而且出问题的往往是生产环境,事实上platform和payment组都出现了这种问题.如果我们不做什么,相信问题还会继续发生.

## Task
虽然我们不能用flyway来做migration了,但是flyway还有validation的功能,所以我们希望可以只用validation的功能.

## Action
1. 我们知道spring-boot提供了丰富的集成配置能力,通过配置可行吗?所以我们马上去查[spring-boot的配置指导](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html),然而很遗憾的是,spring-boot并不能提供通过配置实现我们需求的能力,这种尝试无果而返.
2. 作为一个面向SO编程的程序员,一次尝试的失败难不倒我,很快我们就发现在[spring-boot官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html)里,有这么一句话
> Spring Boot calls Flyway.migrate() to perform the database migration. If you would like more control, provide a @Bean that implements FlywayMigrationStrategy.

Ok,所以我们编写如下一个类
```Java
/**
 * @author Wit
 */
@Configuration
public class Config {

    @Value("${configprofile}")
    String profile;

    @Bean
    public FlywayMigrationStrategy flywayMigrationStrategy() {
        return flyway -> {
            if(!Objects.equals(profile, "dev")
                && !Objects.equals(profile, "sit")
                && !Objects.equals(profile, "uat")) {
                flyway.validate();
            } else {
                flyway.migrate();
            }
        };
    }

}
```
## Result
测试一下,成功.
