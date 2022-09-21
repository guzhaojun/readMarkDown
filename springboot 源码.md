

主要加载的资源类是在哪里获取的

### 入口方法
    1. SpringApplication
    2. run()
   

   ### run:
   headless 用作简单的图像处理


**注意**  
1. 升级springboot的组件还存在不兼容的问题
2.  资源文件过滤问题
    使用继承Spring Boot时，如果要使用Maven resource filter过滤资源文件时，资源文件里面的占位符为了使${}和Spring Boot区别开来，此时要用@...@包起来，不然无效。另外，@...@占位符在yaml文件编辑器中编译报错，所以使用继承方式有诸多问题，坑要慢慢趟。


### start 启动器
