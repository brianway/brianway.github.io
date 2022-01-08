---
layout: post
title: 将项目发布到 Maven 中央仓库踩过的坑
date:   2017-05-17 10:39:07 +08:00
category: 其他
tags: [Maven, IntelliJ-IDEA, 安装部署]
comments: true
---

记录第一次将项目发布到 Maven 中央仓库踩过的坑和解决方案。

<!-- more -->

## 大致步骤

1. 注册 Sonatype 的账户。地址：[https://issues.sonatype.org/secure/Signup!default.jspa](https://issues.sonatype.org/secure/Signup!default.jspa)
2. 提交发布申请。创建 Issue 地址：[https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)
  - 项目类型是 `Community Support - Open Source Project Repository Hosting`
  - groupId 对应的域名你需要有所有权
3. 使用 GPG 生成密钥对。下载地址：[https://www.gnupg.org/download/](https://www.gnupg.org/download/)。用得到的指令有如下几条：
  - `gpg --version` 检查安装成功没
  - `gpg --gen-key` 生成密钥对
  - `gpg --list-keys` 查看公钥
  - `gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 公钥ID` 将公钥发布到 PGP 密钥服务器
  - `gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 公钥ID` 查询公钥是否发布成功
4. 配置 Maven。需要修改的 Maven 配置文件包括：`setting.xml`（全局级别）与 `pom.xml`（项目级别）
  - setting.xml：在其中加入 server 信息，包含 Sonatype 账号的用户名与密码
  - pom.xml：在其中配置 profile,包括插件和 `distributionManagement`，。**snapshotRepository 与 repository 中的 id 一定要与 setting.xml 中 server 的 id 保持一致。**
5. 上传构件到 OSS 中。`mvn clean deploy -P release`
6. 在 OSS 中发布构件。进入 [https://oss.sonatype.org/](https://oss.sonatype.org/)，点击"Staging Repositories" -> 在搜索栏输入你的 groupId -> 勾选你的构件并点击 close -> 点击 tab 栏的 release。
7. 通知 Sonatype 的工作人员关闭 issue。

等待审批通过后，就可以在中央仓库中搜索到自己发布的构件了！下面是我在 Maven 中央仓库的构件：

![webporter 发布到 maven 中央仓库](http://blog.qiniu.brianway.site/webporter_maven-release-v1.0.png)


- 搜索地址： [https://search.maven.org/](https://search.maven.org/)
- 中央仓库地址：[http://mvnrepository.com/](http://mvnrepository.com/)
- 我发布的构件地址：[http://mvnrepository.com/artifact/com.github.brianway](http://mvnrepository.com/artifact/com.github.brianway)


## 图文教程参考

主要参考如下三个链接：

>* [将 Smart 构件发布到 Maven 中央仓库 ](https://my.oschina.net/huangyong/blog/226738) by 黄勇
>* [发布Maven构件到中央仓库](https://my.oschina.net/songxinqiang/blog/313226) by 阿信sxq  
>* [将项目发布到Maven中央库](https://my.oschina.net/looly/blog/270767) by 路小磊 (截图详细)


## 遇到的问题

在这个过程中遇到许多奇奇怪怪的问题，下面依次说明

### 插件没找到

在 IntelliJ IDEA 中，pom.xml 里的插件找不到并报红，如下图：

![maven 依赖不识别](http://blog.qiniu.brianway.site/maven_%E6%8F%92%E4%BB%B6%E4%BE%9D%E8%B5%96%E4%B8%8D%E8%AF%86%E5%88%AB-2.jpeg)

自己使用 mvn 指令构建的话，会提示

```
Plugin '''org.apache.maven.plugins:maven-gpg-plugin:1.6''' not found
Inspects a Maven model for resolution problems.
```

两种解决方法：

- 方案一：手动下载

```
mvn dependency:get -DrepoUrl=http://repo.maven.apache.org/maven2/ -Dartifact=org.apache.maven.plugins:maven-gpg-plugin:1.6
```

- 方案二：在 IntelliJ IDEA 中更新 `Indexed Maven Repositories`

步骤： IntelliJ IDEA -> Preferences -> Build,Execution,Deployment -> Build Tools -> Maven -> Repositories -> Remote URL -> Update

这个过程耗时视网速而定，大概 5～10 分钟。

![maven 依赖不识别解决方案](http://blog.qiniu.brianway.site/maven_%E4%BE%9D%E8%B5%96%E4%B8%8D%E8%AF%86%E5%88%AB%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.png)


方案一是参考下面第二个链接；方案二是我自己提出并验证是有效的。

>* [IntelliJ IDEA shows plugin not found](http://stackoverflow.com/questions/24484776/intellij-idea-shows-plugin-not-found)
>* [Maven plugins can not be found in IntelliJ](http://stackoverflow.com/questions/20496239/maven-plugins-can-not-be-found-in-intellij)


### GPG 版本问题

我用的是 mac,下载的是二进制发行包 GnuPG for OS X，然后发现是在 terminal 输入的指令是 `gpg2` 而不是 `gpg`。比如，公钥显示如下：	

```shell
$ gpg2 --list-keys 
/Users/brian/.gnupg/pubring.kbx
-------------------------------
pub   rsa2048 2017-05-10 [SC] [expires: 2019-05-10]
      DBD686EC6F4E34C4096C427506755FE5978EC644
      DBD686EC6F4E34C4096C427506755FE5978EC644
uid                      brianway <weichuyang@163.com>
sub   rsa2048 2017-05-10 [E] [expires: 2019-05-10]
```

但 Maven 里面的 maven-gpg-plugin 插件默认是使用 `gpg` 指令，不是 `gpg2`，所以需要配置 setting.xml 的 profile

```xml
<profiles>
    <profile>
        <id>gpg</id>
        <properties>
            <gpg.executable>gpg2</gpg.executable>
            <gpg.passphrase>mypassphrase</gpg.passphrase>
        </properties>
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>gpg</activeProfile>
</activeProfiles>
```

> 参考：[http://stackoverflow.com/questions/14114528/avoid-gpg-signing-prompt-when-using-maven-release-plugin](http://stackoverflow.com/questions/14114528/avoid-gpg-signing-prompt-when-using-maven-release-plugin)


### 发布出问题

输入 `mvn clean deploy -P release` 后，报错如下：

```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] webporter-parent ................................... SUCCESS [ 11.690 s]
[INFO] webporter-core ..................................... SUCCESS [  5.208 s]
[INFO] webporter-data-elasticsearch ....................... SUCCESS [  2.769 s]
[INFO] webporter-collector-zhihu .......................... FAILURE [  7.889 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 28.069 s
[INFO] Finished at: 2017-05-11T18:11:44+08:00
[INFO] Final Memory: 45M/723M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.sonatype.plugins:nexus-staging-maven-plugin:1.6.7:deploy (injected-nexus-deploy) on project webporter-collector-zhihu: Failed to deploy artifacts: Could not transfer artifact com.github.brianway:webporter-data-elasticsearch:jar:javadoc:1.0-20170511.101142-1 from/to sonatype-nexus-snapshots (https://oss.sonatype.org/content/repositories/snapshots/): Failed to transfer file: https://oss.sonatype.org/content/repositories/snapshots/com/github/brianway/webporter-data-elasticsearch/1.0-SNAPSHOT/webporter-data-elasticsearch-1.0-20170511.101142-1-javadoc.jar. Return code is: 401, ReasonPhrase: Unauthorized. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :webporter-collector-zhihu

```

搞了几个小时，最后发现是 pom.xml 的 `distributionManagement` 中 snapshotRepository 与 repository 中的 id 与 setting.xml 中 server 的 id **不一致** 导致的。因为我的 pom.xml 是模仿的 [webmagic](https://github.com/code4craft/webmagic) 的 pom.xml，而 settings.xml 的 server 配置却是复制的[《将 Smart 构件发布到 Maven 中央仓库》 by 黄勇 ](https://my.oschina.net/huangyong/blog/226738) 中的，结果导致不一样。


我主要从下面这篇文章中找到的灵感 

> [Maven2部署构件到Nexus时出现的Failed to transfer file错误
](http://www.cnblogs.com/chowmin/articles/3930277.html)

### github 上 release

在项目的 pom.xml 中配置 release 的插件

```xml
 <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
    <version>${plugin.release.version}</version>
    <configuration>
        <tagNameFormat>v@{project.version}</tagNameFormat>
        <autoVersionSubmodules>true</autoVersionSubmodules>
    </configuration>
</plugin>
```

使用下面的指令即可直接在 github 远程仓库生成发行版本

```
mvn clean
mvn release:prepare
mvn release:perform
```

我实验的结果是，执行 `mvn release:prepare` 并不会对远程仓库造成影响，再执行 `mvn release:perform` 才会在 git 远程仓库多出两个 commit 和一个 release 版本，查看提交日志 `git log --pretty=oneline`，内容如下：

```
2619dfaafca7a82f70e938f5b6efd6d9f4110554 [maven-release-plugin] prepare for next development iteration
01795a07b793930b74dc0cb5e6ce2b43ee3a5434 [maven-release-plugin] prepare release v1.0
```

> 参考： [maven scm 配置git](http://blog.csdn.net/u012076316/article/details/52174313)

用上面的方法，不另外配置的话，commit message 是由 Maven 插件自动生成的。也可以不使用插件发布，自己 commit 上去，然后在 github 上发布。

>参考： [Creating Releases 创建发布包](https://github.com/waylau/github-help/blob/master/Creating%20Releases%20%E5%88%9B%E5%BB%BA%E5%8F%91%E5%B8%83%E5%8C%85.md)


### 构件没有出现在 Staging Repositories

按照教程里的步骤，上传构件到 OSS 中后，应该先在 https://oss.sonatype.org/ 中，点击"Staging Repositories" -> 在搜索栏输入你的 groupId -> 勾选你的构件并点击 close -> 点击 tab 栏的 release。

然而我却没有这个步骤，即我上传构件成功后，在 Staging Repositories 中并没有找到自己的构件，但在左边侧栏的 Artifact Search 框输入自己的 groupId 却能搜到我的构件，百思不得其解。最后还是求助 Sonatype 的工作人员才弄清楚，原来是我使用了插件 nexus-staging-maven-plugin 并且默认 autoReleaseAfterClose 是 true 导致的，直接越过了手工 close 的步骤。

Sonatype 工作人员的答复：

> It sounds like you have the nexus-staging-maven-plugin in your build (assuming you're using Maven, but what I'm going to say might also apply to other tools) configured to autoReleaseAfterClose.  If this is the case, oss.sonatype.org will automatically release your staging repository after it has been successfully closed.  oss.sonatype.org is also configured, by default, to drop released staging repositories.  Once your artifacts have been released, they will appear in the Releases repository on oss.sonatype.org, and from there they will sync to Maven Central.
You shouldn't expect to see staging repositories if everything worked and if all your components passed the ruleset validations.  The fact that you can search for your artifacts on search.maven.org is a good sign.

具体可以参看我提的 issue: [OSSRH-31186](https://issues.sonatype.org/browse/OSSRH-31186)


## 参考链接

>* [将 Smart 构件发布到 Maven 中央仓库 ](https://my.oschina.net/huangyong/blog/226738) by 黄勇
>* [发布Maven构件到中央仓库](https://my.oschina.net/songxinqiang/blog/313226) by 阿信sxq  
>* [将项目发布到Maven中央库](https://my.oschina.net/looly/blog/270767) by 路小磊 (截图详细)
>* [GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)
>* [GnuPG的使用入门](http://www.cnblogs.com/buchiany/archive/2017/04/04/6665637.html)（gpg2）
>* [Working with PGP Signatures](http://central.sonatype.org/pages/working-with-pgp-signatures.html)
>* [Creating Releases 创建发布包](https://github.com/waylau/github-help/blob/master/Creating%20Releases%20%E5%88%9B%E5%BB%BA%E5%8F%91%E5%B8%83%E5%8C%85.md)
>* [Sonatype OSSRH-31186](https://issues.sonatype.org/browse/OSSRH-31186)
