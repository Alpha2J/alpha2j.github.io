---
title: 如何创建自己的maven archetype
date: 2023-07-26
tags: 
  - tutorial
  - maven
categories: technology
keywords: 'maven'
---

# 步骤

1. 创建archetype的项目目录，一个archetype也是一个artifact

2. 按照如下要求创建子目录：

   ![image-20240118175846839](../assets/post_img/image-20240118175846839.png)

3. 为archetype创建pom.xml

   ```xml
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
     <modelVersion>4.0.0</modelVersion>
    
     <groupId>my.groupId</groupId>
     <artifactId>my-archetype-id</artifactId>
     <version>1.0-SNAPSHOT</version>
     <packaging>maven-archetype</packaging>
    
     <build>
       <extensions>
         <extension>
           <groupId>org.apache.maven.archetype</groupId>
           <artifactId>archetype-packaging</artifactId>
           <version>3.1.1</version>
         </extension>
       </extensions>
     </build>
   </project>
   ```

4. 创建archetype-metadata.xml

   ```xml
   <archetype-descriptor
           xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0 https://maven.apache.org/xsd/archetype-descriptor-1.1.0.xsd"
           name="quickstart">
       <fileSets>
           <fileSet filtered="true" packaged="true">
               <directory>src/main/java</directory>
           </fileSet>
           <fileSet>
               <directory>src/test/java</directory>
           </fileSet>
       </fileSets>
   </archetype-descriptor>
   ```

5. 创建相应的项目原型文件，以及对应的项目原型pom.xml

   ```xml
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
    
       <groupId>${groupId}</groupId>
       <artifactId>${artifactId}</artifactId>
       <version>${version}</version>
       <packaging>jar</packaging>
    
       <name>${artifactId}</name>
       <url>http://www.myorganization.org</url>
    
       <dependencies>
           <dependency>
                   <groupId>junit</groupId>
                   <artifactId>junit</artifactId>
                    <version>4.12</version>
                   <scope>test</scope>
           </dependency>
       </dependencies>
   </project>
   ```

6. `mvn install`到本地，然后就可以使用这个archetype了：

   ```shell
   mvn archetype:generate                                  \
     -DarchetypeGroupId=<archetype-groupId>                \
     -DarchetypeArtifactId=<archetype-artifactId>          \
     -DarchetypeVersion=<archetype-version>                \
     -DgroupId=<my.groupid>                                \
     -DartifactId=<my-artifactId>
   ```

7. 如果想再使用mvn archetype:generate的时候携带自定义参数，需要使用到`requiredProperties`属性，如：

   ```xml
   <requiredProperties>
     <requiredProperty key="projectName">
       <defaultValue>default project name</defaultValue>
     </requiredProperty>
   </requiredProperties>
   ```

# 参考资料

1. https://maven.apache.org/guides/mini/guide-creating-archetypes.html
1. https://maven.apache.org/archetype/maven-archetype-plugin/create-from-project-mojo.html
1. https://juejin.cn/post/7173891295830605861?searchId=202401181136026CCAA57A05FE498E5C6A
1. https://github.com/ikos23/maven-archetype-sample

