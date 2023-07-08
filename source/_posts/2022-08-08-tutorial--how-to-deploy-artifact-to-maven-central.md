---
title: 如何将artifact发布到Maven中央仓库
date: 2022-08-08
tags: 
  - tutorial
  - maven
categories: technology
keywords: 'maven java'
---

# 1. 背景
最近写了一个[分布式锁组件](https://github.com/hellooo-stack/hellooo-distributedlock)，并在另一个项目中引用了该组件。由于组件lib没有发布到远程仓库，导致每次环境变更，我都要将该分布式组件代码拉下来，然后install到本地Maven仓库，非常麻烦。因此，趁周末决定学习下如何发布到远程的中央仓库，并通过该博文来对整个过程做一个分享。

# 2. 正文
要将artifact发布到Maven中央仓库，只需要按以下4个步骤走：
1. 在[sonatype.org](https://issues.sonatype.org/secure/Signup!default.jspa)上注册账号
2. 申请pom.xml内使用的groupId的所有权，申请地址点[这里](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)
3. 构建项目并部署到“OSSRH Nexus Repository Manager”
4. 发布版本到中央仓库

## 2.1 注册账号
1. 点击[sonatype.org](https://issues.sonatype.org/secure/Signup!default.jspa)进入账号注册页面并填写你的信息：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/384925bfa05042e2ac98e3477fb6d06c~tplv-k3u1fbpfcp-watermark.image?)

之后会收到一封注册成功的通知邮件：

![a81ae3aef2a3af6b6f314f88d580fce.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4ced9bf28694da5a1f6f8daaa9706e6~tplv-k3u1fbpfcp-watermark.image?)

## 2.2 申请groupId的所有权
1. 在能够将指定groupId的artifact发布到中央仓库前，必须申请获得groupId的所有权。申请表单链接在[这里](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)，表单内容可按如下填写：

![d29d8dfdaf165b50b0bd602503c878e.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edb43d20d47d432e85b5ed577e28e06d~tplv-k3u1fbpfcp-watermark.image?)

2. 表单提交后会生成一个issue页面，几分钟后刷新页面，会看到有机器人回复：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cedb13a82c694cd9a6035098f4d4f674~tplv-k3u1fbpfcp-watermark.image?)

主要内容是让你加一条解析记录到正在申请所有权的域名下，比如我的域名注册商是腾讯云，以下是我按要求添加的解析记录：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17209384633b4e62aa4b3da8e774f5ce~tplv-k3u1fbpfcp-watermark.image?)

3. DNS解析记录添加完毕，回到issue界面，回复一个内容，表单状态会自动由"Waiting for response"变回"open"：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd87e641e4bb4496885e5b4b642afed4~tplv-k3u1fbpfcp-watermark.image?)

几分钟后能收到groupId授权成功的回复，此时groupId的已经正式可用：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20e5871cba914ea4911d7e72ce3d7fe3~tplv-k3u1fbpfcp-watermark.image?)

## 2.3 构建项目并部署到“OSSRH Nexus Repository Manager”
1. 在settings.xml中配置远程仓库服务器信息：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06af430753f64f41accae1b6d3ea1989~tplv-k3u1fbpfcp-watermark.image?)

2. 在项目的pom.xml中配置使用的远程仓库信息：
```
    <distributionManagement>
        <snapshotRepository>
<!--            对应settings.xml内配置的<server>下的id -->
            <id>ossrh</id>
            <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>ossrh</id>
            <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
```


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbedd1be11844d05832b0343e99ace3e~tplv-k3u1fbpfcp-watermark.image?)

3. 在项目的pom.xml中配置需要用到的插件信息：
```
<profiles>
    <profile>
        <id>release</id>
        <build>
            <plugins>
                <!--            生成源码 jar -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>2.2.1</version>
                    <executions>
                        <execution>
                            <id>attach-sources</id>
                            <goals>
                                <goal>jar-no-fork</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <!--            生成javadoc jar -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <version>2.9.1</version>
                    <executions>
                        <execution>
                            <id>attach-javadocs</id>
                            <goals>
                                <goal>jar</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <!--            gpg 签名 -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-gpg-plugin</artifactId>
                    <version>1.5</version>
                    <executions>
                        <execution>
                            <id>sign-artifacts</id>
                            <phase>verify</phase>
                            <goals>
                                <goal>sign</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <!--            nexus 组件发布 -->
                <plugin>
                    <groupId>org.sonatype.plugins</groupId>
                    <artifactId>nexus-staging-maven-plugin</artifactId>
                    <version>1.6.7</version>
                    <extensions>true</extensions>
                    <configuration>
                        <serverId>ossrh</serverId>
                        <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
                        <autoReleaseAfterClose>false</autoReleaseAfterClose>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

4. 安装gpg4win，[下载地址](https://www.gpg4win.org/download.html)，使用下图中命令生成key，**生成过程会让输入Passphrase，输入后把它记下来，后面会用到**

![35b9a4a2661692a5adf90bcfa443135.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68eedf7fa6d74ff4978a99390bd5ee60~tplv-k3u1fbpfcp-watermark.image?)


![328d5bf738c5a38b8bbb4f0427c205f.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97543394b3fa4536a53b18b8ad7af976~tplv-k3u1fbpfcp-watermark.image?)

5. 在settings.xml中配置上一步安装的gpg可执行文件的路径以及生成的Passphrase：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73c63da6d8174a5292f78d9e8e3f8bab~tplv-k3u1fbpfcp-watermark.image?)

6. 上传生成的秘钥对的公钥：
```
# 查看当前本机保存的秘钥
gpg --list-signatures --keyid-format 0xshort
# 上传公钥到服务器
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys [你的公钥id]
# 查看公钥是否上传成功
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys [你的公钥id]
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12515050277b4d74ab420612033e091e~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fe1433df20a4ad3b8481e68ac4f8c82~tplv-k3u1fbpfcp-watermark.image?)


7. 在项目的pom.xml文件内添加项目名，项目描述，项目URL等信息，否则，构建会失败，如下图。具体配置可参考[这里](https://github.com/hellooo-stack/hellooo-distributedlock/blob/master/pom.xml)

![016c423d88cf2d6fd82f2ba976b622e.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1862a6ca41664eba834741516e91cb7b~tplv-k3u1fbpfcp-watermark.image?)

8. 执行命令`mvn clean deploy -P release`，如下图所示则表示构建成功，如果构建失败，可以加入`-e`参数查看更多的错误信息，如`mvn clean deploy -P release -e`


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c8e7d4e12df45458f22ebc407ed4d4f~tplv-k3u1fbpfcp-watermark.image?)

9. 完事后，回到你的issue页面刷新，能收到一个机器人的自动回复，意思是发布artifact后，就能在maven的中央仓库中看到我们的artifact了：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7dbea951fc74c159d68a9b0ec8fcafd~tplv-k3u1fbpfcp-watermark.image?)

## 2.4 发布版本到中央仓库
1. 点击[此链接](https://s01.oss.sonatype.org)登录Nexus查看构建结果:

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c467569eef8d439e9c2f8d74809a5587~tplv-k3u1fbpfcp-watermark.image?)

2. 发布构建结果

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/131e490fc73f4738b7b9a4c0c54bac47~tplv-k3u1fbpfcp-watermark.image?)

3. 发布后状态变为closed：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76118589eab94d788b2f471dfde09beb~tplv-k3u1fbpfcp-watermark.image?)

4. 等十几分钟，然后到[这里](https://repo1.maven.org/maven2/)可以搜到我们发布的jar，此时整个过程完事：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ef310921c014b00ad185660edb0ed58~tplv-k3u1fbpfcp-watermark.image?)

5. 如果想自动发布，不走2.4中的第2和第3步，那么可以将`nexus-staging-maven-plugin`这个插件的`<autoReleaseAfterClose>`属性改为`true`，此时构建，打包完成后会自动release，无需手动去点：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d298202cb34243c88482b0139fda483e~tplv-k3u1fbpfcp-watermark.image?)


# 3. 参考资料
1. [Deploying to OSSRH with Apache Maven](https://central.sonatype.org/publish/publish-maven/#review-requirements)
2. [Releasing Deployment from OSSRH to the Central Repository](https://central.sonatype.org/publish/release/)
3. [How to publish artifacts on maven central](https://blog.10pines.com/2018/06/25/publish-artifacts-on-maven-central/)
4. 演示的项目地址为：[hellooo-distributedlock](https://github.com/hellooo-stack/hellooo-distributedlock)，一个Redis下的分布式锁组件实现，欢迎star，共同交流学习！



