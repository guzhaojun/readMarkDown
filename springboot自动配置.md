  使用注解 @EnableAutoConfiguration

  总结
1）通过注解@SpringBootApplication=>@EnableAutoConfiguration=>@Import({AutoConfigurationImportSelector.class})实现自动装配

2）AutoConfigurationImportSelector类中重写了ImportSelector中selectImports方法，批量返回需要装配的配置类

3）通过Spring提供的SpringFactoriesLoader机制，扫描classpath下的META-INF/spring.factories文件，读取需要自动装配的配置类

4）依据条件筛选的方式，把不符合的配置类移除掉，最终完成自动装配



SpringBoot自动装配流程图
![图片](/Users/gzj/Documents/readMarkDown/自动装配图.png)

![图片](http://baidu.com/hellotest.png)