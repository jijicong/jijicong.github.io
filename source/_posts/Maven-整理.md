---
title: Maven 整理
date: 2018-11-30 11:41:42
tags: maven
categories: 工具
---

坐标与依赖
=========

> 了能够自动化地解析任何一个Java构件, Maven必须将它们**唯一**标识, 这就是依赖管理的底层基础-**坐标**.

``` xml
<groupId>org.apache.commons</groupId>
<artifactId>commons-lang3</artifactId>
<version>3.4</version>
<packaging>jar</packaging>
```
元素  |  描述 |  ext
--|---|--
groupId  |  定义当前模块隶属的实际Maven项目, 表示方式与Java包类似 |  groupId不应直接对应项目隶属的公司/组织(一个公司/组织下可能会有很多的项目)
artifactId  |  定义实际项目中的一个Maven模块 |  推荐使用项目名作为artifactId前缀, 如:commons-lang3以commons作为前缀(因为Maven打包默认以artifactId作为前缀)
version  | 定义当前项目所处版本(如1.0-SNAPSHOT、4.2.7.RELEASE、1.2.15、14.0.1-h-3 等)  |  Maven版本号定义约定: <主版本>.<次版本>.<增量版本>-<里程碑版本>
packaging  |定义Maven项目打包方式, 通常打包方式与所生成构件扩展名对应   |  有jar(默认)、war、pom、maven-plugin等.
classifier  | 用来帮助定义构建输出的一些附属构件(如javadoc、sources)  |  不能直接定义项目的classifier(因为附属构件不是由项目默认生成, 须有附加插件的帮助)
<!-- more -->

> 依赖调节原则: 1. 路径最近者优先; 2. 第一声明者优先.更多传递依赖信息可参考:[Dependency Mechanism-Transitive Dependencies][1].

``` xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>4.2.7.RELEASE</version>

    <type>jar</type>
    <scope>compile</scope>
    <optional>false</optional>
    <exclusions>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

元素  | 描述  |  ext
--|---|--
groupId、artifactId和version  | 依赖的基本坐标(最最重要)  |
type  |  依赖的类型, 对应于项目坐标定义的packaging |  默认jar
scope  | 依赖的范围, 用来控制依赖与三种classpath(编译classpath、测试classpath、运行classpath)的关系  |  包含compile、provided、runtime、test、system和import 6种 详见:[Dependency Scope][2]
optional  | 依赖是否可选  |  假如一个Jar包支持MySQL与Oracle两种DB, 因此其构建时必须添加两类驱动, 但用户使用时只会选择一种DB. 此时对A、B就可使用optional可选依赖
exclusions  |  排除传递性依赖 |  传递性依赖极大的简化了项目依赖的管理, 但也会引入Jar包版本冲突等问题, 此时就需要对传递性依赖进行排除.optional与exclusions详细可参考: [Optional Dependencies and Dependency Exclusions][3]

- 依赖管理

  Maven提供了dependency插件可以对Maven项目依赖查看以及优化, 如`$ mvn dependency:tree`可以查看当前项目的依赖树, 详见`$ mvn dependency:help`

依赖范围**scop**e用来控制依赖和编译，测试，运行的**classpath**的关系. 主要的是三种依赖关系如下：
* **compile**： 编译依赖范围。如果没有指定，就会默认使用该依赖范围。使用此依赖范围的Maven依赖，对于编译、测试、运行三种classpath都有效。典型的例子是spring-code,在编译、测试和运行的时候都需要使用该依赖。
* **test**：测试依赖范围。使用次依赖范围的Maven依赖，只对于测试classpath有效，在编译主代码或者运行项目的使用时将无法使用此依赖。典型的例子是Jnuit,它只有在编译测试代码及运行测试的时候才需要。
* **provided**：已提供依赖范围。使用此依赖范围的Maven依赖，对于编译和测试classpath有效，但在运行时候无效。典型的例子是servlet-api,编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器以及提供，就不需要Maven重复地引入一遍。
* **runtime**:运行时依赖范围。使用此依赖范围的Maven依赖，对于测试和运行classpath有效，但在编译主代码时无效。典型的例子是JDBC驱动实现，项目主代码的编译只需要JDK提供的JDBC接口，只有在执行测试或者运行项目的时候才需要实现上述接口的具体JDBC驱动。
* **system**:系统依赖范围。该依赖与三种classpath的关系，和provided依赖范围完全一致，但是，使用system范围的依赖时必须通过systemPath元素显示地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能构成构建的不可移植，因此应该谨慎使用。systemPath元素可以引用环境变量，如：
``` xml
<dependency>
    <groupId>javax.sql</groupId>
    <artifactId>jdbc-stdext</artifactId>
    <Version>2.0</Version>
    <scope>system</scope>
    <systemPath>${java.home}/lib/rt.jar</systemPath>
</dependency>
```
* **import**:导入依赖范围。该依赖范围不会对三种classpath产生实际的影响。
上述除import以外的各种依赖范围与三种classpath的关系如下:

![import][11]

仓库
=========

> Maven 中, 任何一个依赖、插件或项目构建的输出, 都可称为构件, 而Maven仓库就是集中存储这些构件的地方.

Maven仓库可简单分成两类: **本地仓库**与**远程仓库**. 当Maven根据坐标寻找构件时, 它会首先检索本地仓库, 如果本地存在则直接使用, 否则去远程仓库下载.

* 本地仓库:默认地址为**~/.m2/**, 一个构件只有在本地仓库存在之后, 才能由Maven项目使用.

* 远程仓库: 远程仓库又可简单分成两类: **中央仓库**和**私服**.
由于原始的本地仓库是空的, Maven必须至少知道一个远程仓库才能在执行命令时下载需要的构件, 中央仓库就是这样一个**默认的远程仓库**.

> 关于仓库的详细配置可参考: [Settings Reference][4] 的 *Servers、Mirrors、Profiles* 等章节.

``` xml
<project >
    ...
    <distributionManagement>
        <repository>
            <id>releases</id>
            <url>http://mvn.server.com/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://mvn.server.com/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

</project>
```
* `repository`表示**发布版本**构件的仓库, `snapshotRepository`代表快照版本的仓库.
* `id`为该远程仓库唯一标识, `url`表示该仓库地址.

配置正确后, 执行``$ mvn clean deploy``则可以将项目构建输出的构件部署到对应配置的远程仓库.

> 注: 往远程仓库部署构件时, 往往需要认证: 需要在*settings.xml*中创建**servers**元素, 其id与仓库的id匹配, 详见: [Password Encryption][5].

生命周期与插件
=========

> Maven 将所有项目的构建过程统一抽象成一套生命周期: 项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成 … 几乎所有项目的构建,都能映射到这一组生命周期上. 但生命周期是抽象的(Maven的生命周期本身是不做任何实际工作), 任务执行(如编译源代码)均交由插件完成. 其中每个构建步骤都可以绑定一个或多个插件的目标,而且Maven为大多数构建步骤都编写并绑定了默认插件.当用户有特殊需要的时候, 也可以配置插件定制构建行为, 甚至自己编写插件.

Maven 拥有三套相互独立的生命周期:*** clean***、***default*** 和 ***site***, 而每个生命周期包含一些phase阶段, 阶段是有顺序的, 并且后面的阶段依赖于前面的阶段. 而三套生命周期相互之间却并没有前后依赖关系, 即调用site周期内的某个phase阶段并不会对clean产生任何影响.

* clean

  clean生命周期的目的是清理项目:

  执行如``$ mvn clean``;

* default

  default生命周期定义了真正构建时所需要执行的所有步骤:

  执行如``$ mvn clean install``;

* site

  site生命周期的目的是建立和发布项目站点: Maven能够基于POM所包含的信息,自动生成一个友好的站点,方便团队交流和发布项目信息

  执行命令如``$ mvn clean deploy site-deploy``;

> 更多 Maven 生命周期介绍可以参考: [Introduction to the Build Lifecycle][6]、[Lifecycles Reference][7].

生命周期的阶段**phase**与插件的目标**goal**相互绑定, 用以完成实际的构建任务. 而对于插件本身, 为了能够复用代码,它往往能够完成多个任务, 这些功能聚集在一个插件里,每个功能就是一个目标.
如:``$ mvn compiler:compile``冒号前是插件前缀, 后面是该插件目标(即: **maven-compiler-plugin**的**compile**目标).

为了能让用户几乎不用任何配置就能使用Maven构建项目, Maven 默认为一些核心的生命周期绑定了插件目标, 当用户通过命令调用生命周期阶段时, 对应的插件目标就会执行相应的逻辑.

* clean生命周期阶段绑定

生命周期阶段  |  插件目标
--|--
pre-clean  |  -
clean  |  maven-clean-plugin:clean
post-clean  |  -

* default声明周期阶段绑定

生命周期阶段  |  插件目标 |  执行任务
--|---|--
process-resources  | maven-resources-plugin:resources  |  复制主资源文件到主输出目录
compile  |  maven-compiler-plugin:compile |  编译主代码到主输出目录
process-test-resources  | maven-resources-plugin:testResources  |  复制测试资源文件到测试输出目录
test-compile  |  maven-compiler-plugin:testCompile |  编译测试代码到测试输出目录
test  | maven-surefire-plugin:test  |  执行测试用例
package  |  maven-jar-plugin:jar |  打jar包
install  |  maven-install-plugin:install |  将项目输出安装到本地仓库
deploy  | maven-deploy-plugin:deploy  |  	将项目输出部署到远程仓库

* site生命周期阶段绑定

生命周期阶段  |  插件目标
--|---
pre-site  |    -
site  |  maven-site-plugin:site
post-site  |  -
site-deploy  |  maven-site-plugin:deploy

除了内置绑定以外, 用户还能够自定义将某个插件目标绑定到生命周期的某个阶段上. 如创建项目的源码包, *maven-source-plugin*插件的*jar-no-fork*目标能够将项目的主代码打包成jar文件, 可以将其绑定到verify阶段上:

``` xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

* `executions`下每个`execution`子元素可以用来配置执行一个任务.

> 关于Maven插件的自定义绑定及其他详细配置可参考: [Guide to Configuring Plug-ins][8].

模块聚合
=========

> Maven的聚合特性(*aggregation*)能够使项目的多个模块聚合在一起构建, 而继承特性(*inheritance*)能够帮助抽取各模块相同的依赖、插件等配置,在简化模块配置的同时, 保持各模块一致.

* 聚合POM

  聚合模块POM仅仅是帮助聚合其他模块构建的工具, 本身并无实质内容:

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.vdian.feedcenter</groupId>
    <artifactId>feedcenter-push</artifactId>
    <packaging>pom</packaging>
    <version>1.0.0.SNAPSHOT</version>

    <modules>
        <module>feedcenter-push-client</module>
        <module>feedcenter-push-core</module>
        <module>feedcenter-push-web</module>
    </modules>

</project>
```

* 通过在一个打包方式为`pom`的Maven项目中声明任意数量的`module`以实现模块聚合:
  1. `packaging`: `pom`, 否则无法聚合构建.
  2. `modules`: 实现聚合的核心,`module`值为被聚合模块相对于聚合POM的**相对路径**, 每个被聚合模块下还各自包含有*`pom.xml`*、*`src/main/java`*、*`src/test/java`*等内容, 离开聚合POM也能够独立构建(注: 模块所处目录最好与其***artifactId***一致).

> Tips: 推荐将聚合POM放在项目目录的最顶层, 其他模块作为聚合模块的子目录.
> 其他关于聚合与反应堆介绍可参考: [Guide to Working with Multiple Modules][9].

模块继承
=========

在面向对象中, 可以通过类继承实现复用. 在Maven中同样也可以创建POM的父子结构, 通过在父POM中声明一些配置供子POM继承来实现复用与消除重复:

与聚合类似, 父POM的打包方式也是pom, 因此可以继续复用聚合模块的POM(这也是在开发中常用的方式):

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.vdian.feedcenter</groupId>
    <artifactId>feedcenter-push</artifactId>
    <packaging>pom</packaging>
    <version>1.0.0.SNAPSHOT</version>

    <modules>
        <module>feedcenter-push-client</module>
        <module>feedcenter-push-core</module>
        <module>feedcenter-push-web</module>
    </modules>

    <properties>
        <finalName>feedcenter-push</finalName>
        <warName>${finalName}.war</warName>
        <spring.version>4.0.6.RELEASE</spring.version>
        <junit.version>4.12</junit.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <warExplodedDirectory>exploded/${warName}</warExplodedDirectory>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>

           <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>3.0.0</version>
                    <executions>
                        <execution>
                            <id>attach-sources</id>
                            <phase>verify</phase>
                            <goals>
                                <goal>jar-no-fork</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

* `dependencyManagement`: 能让子POM继承父POM的配置的同时, 又能够保证子模块的灵活性: 在父POM `dependencyManagement`元素配置的依赖声明不会实际引入子模块中, 但能够约束子模块`dependencies`下的依赖的使用(子模块只需配置groupId与artifactId, 见下).

* `pluginManagement`: 与`dependencyManagement`类似, 配置的插件不会造成实际插件的调用行为, 只有当子POM中配置了相关`plugin`元素, 才会影响实际的插件行为.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <parent>
        <groupId>com.vdian.feedcenter</groupId>
        <artifactId>feedcenter-push</artifactId>
        <version>1.0.0.SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>feedcenter-push-client</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

* 元素继承

  可以看到, 子POM中并未定义模块groupId与version, 这是因为子POM默认会从父POM继承了如下元素:
  * groupId、version
  * dependencies
  * developers and contributors
  * plugin lists (including reports)
  * plugin executions with matching ids
  * plugin configuration
  * resources

* 优势: 当依赖、插件的版本、配置等信息在父POM中声明之后, 子模块在使用时就无须声明这些信息, 也就不会出现多个子模块使用的依赖版本不一致的情况, 也就降低了依赖冲突的几率. 另外如果子模块不显式声明依赖与插件的使用, 即使已经在父POM的`dependencyManagement`、`pluginManagement`中配置了, 也不会产生实际的效果.

* 推荐: 模块继承与模块聚合同时进行,这意味着, 你可以为你的所有模块指定一个父工程, 同时父工程中可以指定其余的Maven模块作为它的聚合模块. 但需要遵循以下三条规则:
  - 在所有子POM中指定它们的父POM;
  - 将父POM的`packaging`值设为`pom`;
  - 在父POM中指定子模块/子POM的目录.

> 注: parent元素内还包含一个relativePath元素, 用于指定父POM的相对路径, 默认../pom.xml.

任何一个Maven项目都隐式地继承自超级POM, 因此超级POM的大量配置都会被所有的Maven项目继承, 这些配置也成为了Maven所提倡的约定.

``` xml
<!-- START SNIPPET: superpom -->
<project>
  <modelVersion>4.0.0</modelVersion>

  <!-- 定义了中央仓库以及插件仓库, 均为:https://repo.maven.apache.org/maven2 -->
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>

  <!-- 依次定义了各类代码、资源、输出目录及最终构件名称格式, 这就是Maven项目结构的约定 -->
  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>

    <!-- 为核心插件设定版本 -->
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.3.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

  <!-- 定义项目报告输出路径 -->
  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>

  <!-- 定义release-profile, 为构件附上源码与文档 -->
  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>

      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>

      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>

</project>
<!-- END SNIPPET: superpom -->
```

> 附: Maven继承与组合的其他信息还可参考: [Introduction to the POM][10].

[1]:  https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies
[2]: https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope
[3]: https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html
[4]: https://maven.apache.org/settings.html
[5]: https://maven.apache.org/guides/mini/guide-encryption.html#How_to_encrypt_server_passwords
[6]: https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
[7]: https://maven.apache.org/ref/3.3.9/maven-core/lifecycles.html
[8]: https://maven.apache.org/guides/mini/guide-configuring-plugins.html
[9]: https://maven.apache.org/guides/mini/guide-multiple-modules.html
[10]: https://maven.apache.org/guides/introduction/introduction-to-the-pom.html
[11]: /Maven-整理/import.png
