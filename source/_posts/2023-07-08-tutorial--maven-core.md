---
title: maven核心概念的一次探索
date: 2023-07-08
tags: 
  - tutorial
  - maven
categories: technology
keywords: 'maven'
---

# 1. 背景

使用Maven已经好多年，最常用也最熟悉的就是它的依赖管理：什么增删改dependency，依赖冲突处理啥的，对一些比较少用的功能理解程度还是不够。这两天在搭建本地的nexus3私服，发现有些概念如果不彻底搞明白，会很影响我整个项目的搭建速度以及搭建出来的服务的可靠性，因此有了这篇文章。如果大家有以下的和我一样的疑惑，那么希望本篇文章能够帮助到你：

1. maven的仓库体系到底是怎样的：怎么定义仓库、有哪些仓库、怎么配置仓库、什么是镜像仓库？
2. 构件的上传下载规则是怎样的：从仓库下载构建的规则？、上传构建到仓库的规则？
3. maven区分构件版本的规则是怎样的：怎么区分哪些是snapshot构件，哪些是releases构件
4. 依赖的scope怎么理解：什么是scope、有哪些类型的scope、不同scope的作用分别是什么
5. nexus3的仓库类型：有哪些、规则分别是什么
6. 聚合与继承怎么理解
7. 冲突处理规则是怎样的
8. 什么是可选依赖

# 2. 正文

## 2.1 maven的仓库体系

1. 怎么定义仓库：maven构建出来的东西就叫做“构件”，就是平时用到的各种jar包或者war包等，如果要把这些构件共享给别人用，那么就要有某个地方来存放它们，而仓库就是用来存放它们的地方。简单来说就是存取构件的一个地方。
2. 有哪些仓库：首先，maven中只有本地仓库和远程仓库；本地仓库就是`.m2/repository`这个目录，而远程仓库则是网络上运行的maven仓库服务，有很多很多，如maven的中央仓库[https://mvnrepository.com/](https://mvnrepository.com/)，google的[https://maven.google.com/web/index.html](https://maven.google.com/web/index.html)，等等。
    - 对于远程仓库，默认的maven中央仓库的id为：central
3. 怎么配置仓库：主要涉及到两个地方的配置。上传构件时用到的服务器的鉴权信息，在settings.xml内配置；下载构件时远程服务器的元数据信息，在pom.xml内配置。如：
   
    ```xml
    <!-- settings.xml配置仓库服务器鉴权信息 -->
    <servers>
      <server>
        <id>devenv-public</id>
        <username>admin</username>
        <password>f9f5c64c-9ba2-4396-85c7-c2d2da4898cd</password>
      </server>
    </servers>
    
    <!-- pom.xml配置远程仓库信息 -->
    <repository>
      <id>devenv-public</id>
      <url>http://devenv.local:17000/repository/maven-public/</url>
    </repository>
    ```
    
4. 什么是镜像仓库：如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。也就是说，可以从镜像仓库中获取到被镜像仓库的所有内容，一般用来加速仓库的访问，比如阿里镜像仓库的配置：[https://developer.aliyun.com/mirror/maven/](https://developer.aliyun.com/mirror/maven/)
   
    ```xml
    <mirror>
        <id>aliyunmaven</id>
        <!-- 意思是镜像中央仓库，所有对中央仓库的请求都会发送到这个仓库 -->
        <mirrorOf>central</mirrorOf>
        <name>aliyunpublic</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
    ```
    

## 2.2 构件的上传下载规则

1. 从仓库下载构建的规则：
    - 首先，构件是从仓库下载的，得先配置仓库：
        - 本地仓库：默认位于`～/.m2/repository`，可在settings.xml中配置
        - 远程仓库：
            - maven lib中，写死的中央仓库路径
            - 项目的`pom.xml`中，`<repositories>`标签内配置的仓库。示例及解释如下：
              
                ```xml
                <repositories>
                <!--    <snapshots/>和<releases>的enabled默认都是true，所以对于snapshot和release的构件都可以从这个仓库下载   -->
                        <repository>
                            <id>devenv-public</id>
                            <url>http://devenv.local:17000/repository/maven-public/</url>
                        </repository>
                
                <!--    <snapshots/>的enabled为false，所以对于一个snapshot构件，不会从这个仓库下载   -->
                        <repository>
                            <id>devenv-release</id>
                            <url>http://devenv.local:17000/repository/maven-releases/</url>
                            <snapshots>
                                <enabled>false</enabled>
                            </snapshots>
                        </repository>
                
                <!--    <releases/>的enabled为false，所以对于一个release构件，不会从这个仓库下载   -->
                        <repository>
                            <id>devenv-snapshot</id>
                            <url>http://devenv.local:17000/repository/maven-snapshots/</url>
                            <releases>
                                <enabled>false</enabled>
                            </releases>
                        </repository>
                    </repositories>
                ```
        
    - mvn构建项目时，能感知到所有的这些仓库，然后按照如下顺序拉取构件：本地仓库 → 远程仓库（依次访问项目pom.xml中配置的仓库） → 远程仓库（中央仓库）
    - 例子：针对如下的pom.xml文件配置，下载`commons-codec`时的规则为：先从本地仓库查找groupId=commons-codec, artifactId=commons-codec, version=1.16.0的jar；如果找不到，会到第一个支持下载release包的仓库查找，这里是`devenv-public`；如果找不到，那么会到第二个支持release包的仓库查找，这里是`devenv-release`；如果还是找不到，会到maven lib中配置的默认中央仓库中查找，最终如果找不到，那么会报错。
      
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
            <modelVersion>4.0.0</modelVersion>
        
            <groupId>org.example</groupId>
            <artifactId>testmaven</artifactId>
            <version>1.0.0-snapshot</version>
        
            <dependencies>
                <dependency>
                    <groupId>commons-codec</groupId>
                    <artifactId>commons-codec</artifactId>
                    <version>1.16.0</version>
                </dependency>
            </dependencies>
        
            <repositories>
        <!--    <snapshots/>和<releases>的enabled默认都是true，所以对于snapshot和release的构件都可以从这个仓库下载   -->
                <repository>
                    <id>devenv-public</id>
                    <url>http://devenv.local:17000/repository/maven-public/</url>
                </repository>
        
        <!--    <snapshots/>的enabled为false，所以对于一个snapshot构件，不会从这个仓库下载   -->
                <repository>
                    <id>devenv-release</id>
                    <url>http://devenv.local:17000/repository/maven-releases/</url>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
        
        <!--    <releases/>的enabled为false，所以对于一个release构件，不会从这个仓库下载   -->
                <repository>
                    <id>devenv-snapshot</id>
                    <url>http://devenv.local:17000/repository/maven-snapshots/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                </repository>
            </repositories>
        </project>
        ```
        
    - 对于上一步中的“例子”，为什么不会到`devenv-snapshot`这个仓库查找呢，因为这个仓库配置了`<releases><enabled>false</enabled></releases>`，关闭了release包的支持，只支持snapshot包，而这里的`1.16.0`很明显是一个release版本的包。
2. 上传构建到仓库的规则：规则很简单，首先需要在项目`pom.xml`中使用`<distributionManagement/>`支持的上传仓库，然后需要指定当前项目的artifact版本，上传的时候会根据版本的类型不同（release还是snapshot）上传到不同的仓库。如snapshot包`1.0.0-snapshot`只能上传到支持snapshot包的仓库，release包`1.0.0`只能上传到支持release包的仓库。
   
    ```xml
    <distributionManagement>
      <repository>
        <id>devenv-release</id>
        <url>http://devenv.local:17000/repository/maven-releases/</url>
      </repository>
      <snapshotRepository>
        <id>devenv-snapshot</id>
        <url>http://devenv.local:17000/repository/maven-snapshot/</url>
      </snapshotRepository>
    </distributionManagement>
    ```
    

## 2.3 maven区分构件版本的规则

- 概念：maven中构件的版本分为snapshot和release；在下载构件时会根据待下载构件的版本类型到配置的**仓库列表**中选择支持对应版本的仓库执行下载，在上传构件时，同样会根据待上传构件的版本类型到配置的**上传仓库列表**选择支持对应版本的仓库执行上传。
- 区分规则：maven使用一种特定的命名约定来根据版本号区分是发布版本（release）还是快照版本（snapshot）：
    - 发布版本(Release)：发布版本使用固定的版本号，不包含任何特殊标识符。例如，**`1.0`**、**`2.3.4`** 等都是发布版本号。
    - 快照版本(Snapshot)：快照版本使用带有 **`-SNAPSHOT`** 后缀的版本号。例如，**`1.0-SNAPSHOT`**、**`2.3.4-SNAPSHOT`** 等都是快照版本号。

## 2.4 依赖的scope

概念：maven在编译项目主代码、运行项目项目主代码、编译和执行测试代码时都会分别有一套classpath，依赖的jar会被放入对应的classpath中。而scope就是用来控制使用哪个classpath的。有如下的几种scope：

- compile: 编译，运行，测试时都有效，就是说在三套classpath下都存在该构件
- runtime: 运行时有效（如jdbc驱动），仅在运行时的classpath中存在该构件
- test: 仅在测试时（编译测试代码和运行测试）有效
- provided: 编译主代码和测试时有效，运行时无效（如servlet-api）

特殊的scope，`import`：这个scope只有在`<dependencyManagement/>`元素下使用时才会有效果，通常指向一个pom文件，表示把该文件内的`<dependencyManagement/>`配置导入合并到当前pom的`<dependencyManagement/>`元素中。

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>pr</artifactId>
      <version>1.0.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

## 2.5 nexus3的仓库类型

- 概念：私服，私服也是远程仓库
- nexus3私服仓库主要类型有：group、hosted、proxy
    - proxy：代理仓库充当了远程仓库的缓存，它会在首次访问远程仓库时将所需的组件下载到本地，并在后续的请求中直接从本地提供。这样可以加快构建和部署过程，并减轻对远程仓库的依赖。
    - hosted：托管仓库是用于存储和管理本地组件的地方。开发人员可以将自己的软件包、库或插件发布到托管仓库中，供团队内部或公共访问。这种仓库类型使得组件的共享和分发变得更加方便。
    - group：组合仓库是一个逻辑上的集合，它将多个仓库聚合在一起，并将它们呈现为单个仓库。通过组合仓库，开发人员可以同时从多个仓库中检索组件，而不必直接访问每个仓库。这在多个团队或多个仓库之间共享组件时非常有用。

## 2.6 其他

1. 聚合与继承：两个完全不同的概念。
    - 聚合：对于聚合模块来说，它知道有哪些被聚合的模块，但对于那些被聚合模块来说，它不知道这个聚合模块的存在。**聚合主要为了方便快速构建项目，可以在聚合模块中一次性构建所有被聚合模块。**
    - 继承：对于父pom来说，它不知道有哪些子模块继承于它，而对于子模块来说，它必须知道它的父pom是什么。**继承主要为了消除重复配置，可以在子模块中复用父pom中的配置。**其中relativePath用于表示父pom的位置，默认是`../pom.xml`。
2. 冲突处理：两个原则，最短路径和第一声明者优先
    - 最短路径：A → B → C → X (1.0) 和 A → D → X (2.0) 这里会使用X(2.0)
    - 第一声明者优先：A → B → Y (1.0) 和 A → C → Y (2.0) 这里两者路径长度都一样，如果B的依赖声明在C之前，那么Y (1.0)就会被解析使用。
3. 可选依赖：A → B、B → X（可选）、B → Y（可选），如果三个依赖的范围的是compile，那么X、Y就是A的compile范围传递性依赖。但是这里X和Y都是可选的，所以依赖不会被传递，X和Y对A没有任何影响。

# 3. 参考资料

- Maven实战 许晓斌：[https://item.jd.com/27160441701.html](https://item.jd.com/27160441701.html)
