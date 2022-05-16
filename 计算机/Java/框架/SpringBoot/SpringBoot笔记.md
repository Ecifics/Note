# SpringBoot

## 1. @Bean

一般@Bean用在配置类中，其中@Configuration将这个类标识为配置类

```java
@Configuration
public class MyConfig {
    @Bean
    public User user01() {
        return new User("zhangsan", 18);
    }
}
```



> 注：
>
> @Bean 注解给容器中添加组件，以方法名为组件的id，返回类型就是组件类型，返回的值就是组件在容器中的实例



## 2. @Conditional

### 2.1 @ConditionalOnBean

这个注解表示如果某个bean生效了，才回使当前的bean生效，例如下面代码

```java
@Configuration
public class MyConfig {

    /**
     * @Bean 注解给容器中添加组件，以方法名为组件的id，返回类型就是组件类型
     * @return
     */
    @Bean
    @ConditionalOnBean(name = "tomcat")
    public User user01() {
        return new User("zhangsan", 18, new Pet("tomcat"));
    }

    //@Bean
    public Pet tomcat() {
        return new Pet("tomcat");
    }
}
```

如果tomcat bean没有生效，那么user01组件也不会生效

>
>
>注：
>
>这个注解也可以加在类上，表示如果其name属性中所对应的bean没有生效之前，类中的所有bean都不会生效

### 2.2 @ConditionalOnMissingBean

其中可以指定name属性的值，表示如果name属性对应的bean没有生效，才会使对应的bean或者类中的所有bean生效	