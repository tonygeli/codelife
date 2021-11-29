

常用注解

@EnableAutoConfiguration

@ComponentScan



@Bean

@Controller @Service @ResponseBody @Autowired



#### 如何再SpringBoot启动后运行特定方法

```java
/Users/admin/.m2/repository/org/springframework/boot/spring-boot/2.1.1.RELEASE/spring-boot-2.1.1.RELEASE.jar!/org/springframework/boot/SpringApplication.class:490


public class Application implements ApplicationRunner {
  @Override
	public void run(ApplicationArguments args) throws Exception {}
}

public class Application implements CommandLineRunner {
	@Override
	public void run(String... args) throws Exception {}
}

private void callRunners(ApplicationContext context, ApplicationArguments args) {
  List<Object> runners = new ArrayList();
  runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
  runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
  AnnotationAwareOrderComparator.sort(runners);
  Iterator var4 = (new LinkedHashSet(runners)).iterator();

  while(var4.hasNext()) {
    Object runner = var4.next();
    if (runner instanceof ApplicationRunner) {
      this.callRunner((ApplicationRunner)runner, args);
    }

    if (runner instanceof CommandLineRunner) {
      this.callRunner((CommandLineRunner)runner, args);
    }
  }
}
```





1．掌握SpringBoot与DataSource数据源整合。
2．掌握SpringBoot与MyBatis开发框架整合。
3．掌握SpringBoot与SpringDataJPA开发框架整合。
4．掌握SpringBoot与消息组件（ActiveMQ、RabbitMQ、Kafka）整合。
5．掌握SpringBoot与邮件服务整合。
6．掌握SpringBoot与定时调度服务整合。
7．掌握SpringBoot与Redis数据库整合。
8．掌握SpringBoot与Restful服务整合。

























