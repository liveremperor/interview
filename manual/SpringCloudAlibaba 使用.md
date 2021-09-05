# SpringCloudAlibaba 使用

Spring Cloud Alibaba 致力于提供分布式应用服务开发的一站式解决方案。项目包含开发分布式应用服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

- nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理

## 1. 创建父工程

pom.xml 文件

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>msdemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>msdemo</name>
    <description>Demo project for Spring cloud alibaba</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- boot-version -->
        <spring.boot-version>2.3.0.RELEASE</spring.boot-version>
        <!-- cloud-version -->
        <spring.cloud-version>Hoxton.SR3</spring.cloud-version>
        <!-- cloud.alibaba-version -->
        <spring.cloud.alibaba-version>2.2.1.RELEASE</spring.cloud.alibaba-version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

## 2. 启动nacos

[nacos releases](https://github.com/alibaba/nacos/releases)

最新2.0.3版本下载 [nacos-server-2.0.3.tar.gz](https://github.com/alibaba/nacos/releases/download/2.0.3/nacos-server-2.0.3.tar.gz)  [nacos-server-2.0.3.zip](https://github.com/alibaba/nacos/releases/download/2.0.3/nacos-server-2.0.3.zip)  

1. 解压压缩包

   ``` bash
   tar -xvf nacos-server-2.0.3.tar.gz
   ```

2. 启动服务器

   standalone代表着单机模式运行，非集群模式

   Linux:

   ``` bash
   cd nacos/bin
   sh startup.sh -m standalone
   ```

   Windows:

   ``` bash
   startup.cmd -m standalone
   ```

3. 访问页面

   http://localhost:8848/nacos

   用户名/密码 默认 nacos/nacos

   看到此页面代表启动登陆成功

   ![image-20210905201203467](C:/Users/YU/AppData/Roaming/Typora/typora-user-images/image-20210905201203467.png)

## 3. 服务注册

1. 创建Module，provider-server

   pom.xml 文件

   ``` xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>msdemo</artifactId>
           <groupId>com.example</groupId>
           <version>0.0.1-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>provider-server</artifactId>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           </dependency>
       </dependencies>
   
   </project>
   ```

   bootstrap.yml 配置文件

   ``` yaml
   spring:
     application:
       name: provider-server
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
   server:
     port: 8010
   ```

   ProviderServerApplication.java 主启动类

   ``` java
   package org.example.msdemo;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   @EnableDiscoveryClient
   public class ProviderServerApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ProviderServerApplication.class, args);
       }
   
   }
   ```

   启动服务，查看注册中心，服务provider-server已经注册到nacos

   ![image-20210905203246163](C:/Users/YU/AppData/Roaming/Typora/typora-user-images/image-20210905203246163.png)

## 4. 服务配置

​	在 Nacos Spring Cloud 中，**dataId** 的完整格式如下：	

```properties
${prefix}-${spring.profiles.active}.${file-extension}
```

- **prefix** 默认为 **spring.application.name** 的值，也可以通过配置项 **spring.cloud.nacos.config.prefix** 来配置。
- **spring.profiles.active** 即为当前环境对应的 profile。注意：当 **spring.profiles.active** 为空时，对应的连接符 **-** 也将不存在，**dataId** 的拼接格式变成 **${prefix}.${file-extension}**
- **file-exetension** 为配置内容的数据格式，可以通过配置项 **spring.cloud.nacos.config.file-extension** 来配置。目前只支持 **properties** 和 **yaml** 类型。

1. 在nacos控制台新增配置：

   ![image-20210905211240874](C:/Users/YU/AppData/Roaming/Typora/typora-user-images/image-20210905211240874.png)

2. 修改provider-server服务pom.xml，添加依赖

   ``` xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

3. 修改provider-server服务bootstrap.yml 配置文件，添加配置地址，指定配置类型

   ``` yaml
   spring:
     application:
       name: provider-server
     cloud:
       nacos:
         discovery:
           server-addr: 115.29.174.79:8848
         config:
           server-addr: 115.29.174.79:8848
           # 指定配置类型
           file-extension: yaml
   server:
     port: 8010
   ```

4. 添加控制器

   ``` java
   @RefreshScope
   @RestController
   public class ConfigController {
   
       @Value("${userName:none}")
       private String userName;
   
       @GetMapping("/config")
       public String config() {
           return userName;
       }
   }
   ```

5. 启动服务，访问http://localhost:8010/config，返回配置项