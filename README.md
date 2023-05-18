# 说明

parent pom通常用来定义一些项目所共同遵守的约定，比如一些公共的三方依赖、编码、版本、插件、发布的仓库等等。此项目中定义了一些编码、版本、插件的声明以及发布仓库的声明。

# 使用

- 创建自己的项目或者克隆此项目
- 根据实际需要修改项目的名称、groupId、artifactId、version等
- 将此项目发布到需要的仓库中
- 在其他的项目中指定当前项目为父模块即可

其他项目中的POM文件中引用当前项目为父模块：

```
<parent>
    <groupId>me.cxis</groupId>
    <artifactId>parent-pom</artifactId>
    <version>2.0.0-SNAPSHOT</version>
</parent>
```

# 功能说明以及注意事项

## 编码声明

声明了构建以及报告输出的编码为`UTF-8`

## Java以及编译版本

声明了java的版本以及maven编译的版本为`17`

## maven基础插件的声明

声明并升级了maven基础插件，列表如下：

- `maven-compiler-plugin`：编译插件
- `maven-source-plugin`：源码插件
- `maven-surefire-plugin`：单测插件
- `maven-deploy-plugin`：部署插件

### maven-compiler-plugin

- 升级了`maven-compiler-plugin`插件版本为`3.11.0`
- `maven-compiler-plugin`的java版本省纪委`17`

具体声明如下：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>${maven-compiler-plugin.version}</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
    </configuration>
</plugin>
```

`maven-compiler-plugin`有如下两个目标：

- `compiler:compile`：默认绑定到default生命周期的`compile`阶段
- `compiler:testCompile`：默认绑定到default生命周期的`test-compile`阶段

一般实际使用的时候无需特别指定，在`mvn package`以及往上的阶段执行的时候都会默认执行掉`maven-compiler-plugin`的两个目标，如果需要单独执行compiler的目标，只需要使用如下命令即可：`mvn compile`或者`mvn test-compile`。

### maven-source-plugin

- 升级了`maven-source-plugin`插件版本为`3.2.1`
- 指定了执行的目标为`jar-no-fork`和`test-jar-no-fork`

具体声明如下：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>${maven-source-plugin.version}</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <goals>
                <goal>jar-no-fork</goal>
                <goal>test-jar-no-fork</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

`maven-source-plugin`的`jar`、`test-jar`、`jar-no-fork`、`test-jar-no-fork`四个目标默认绑定到default声明周期的`package`阶段，并且默认执行的目标是`jar`和`test-jar`，由于`jar`和`test-jar`两个目标在执行前会自动执行其他阶段的目标，导致重复执行，故在此将执行目标指定为了：`jar-no-fork`和`test-jar-no-fork`，这样可以防止重复执行其他阶段的目标。

由于默认绑定到了default生命周期的`package`阶段，故在`mvn package`以及以上的阶段执行的时候会自动执行`maven-source-plugin`的目标，如果需要单独执行目标，只需要使用如下命令即可：`mvn source:jar`、`mvn source:test-jar`、`mvn source:jar-no-fork`、`mvn source:test-jar-no-fork`。

#### 使用注意事项

一般情况下只需要将对外提供的api的源码打包并且deploy到仓库中，对于项目中核心实现的代码并不需要打包源码并且deploy到仓库中，可以在实际的实现模块中禁用`maven-source-plugin`插件。使用方式是在核心实现模块的`pom`文件中添加如下方式进行禁用：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <configuration>
        <!-- 指定当前模块不打包源码 -->
        <skipSource>true</skipSource>
    </configuration>
</plugin>
```

### maven-surefire-plugin

- 升级了`maven-surefire-plugin`插件版本为`3.1.0`

具体声明如下：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${maven-surefire-plugin.version}</version>
</plugin>
```

`maven-surefire-plugin`只有一个`surefire:test`目标，默认绑定到default声明周期的`test`阶段，在`mvn test`以及以上的阶段执行的时候都会执行`surefire:test`目标。

#### 使用注意事项

如果需要执行其他的一些不符合默认命名规则的单测类，可以在项目中使用如下配置进行添加：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <includes>
            <!--- 添加spock的单测类 -->
            <include>**/*Spec</include>
            
            <!-- 以下将默认的命名规则添加进来 -->
            <include>**/Test*.java</include>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
            <include>**/*TestCase.java</include>
        </includes>
    </configuration>
</plugin>
```

有些项目中可能会需要指定需要的单测框架的版本，可以在项目中进行配置，比如如下示例是指定`junit4`的依赖：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.apache.maven.surefire</groupId>
            <artifactId>surefire-junit4</artifactId>
            <version>${surefire-junit4.version}</version>
        </dependency>
    </dependencies>
</plugin>
```

### maven-deploy-plugin

- 升级了`maven-deploy-plugin`插件版本为`3.1.1`

具体声明如下：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-deploy-plugin</artifactId>
    <version>${maven-deploy-plugin.version}</version>
</plugin>
```

`maven-deploy-plugin`插件的`deploy`目标默认绑定到default生命周期的`deploy`阶段，需要使用`mvn deploy`命令来进行执行。

#### 使用注意事项

一般情况下只需要将对外提供的api包打包并且deploy到仓库中，对于项目中核心实现的代码并不需要打包并且deploy到仓库中，可以在实际的实现模块中禁用`maven-deploy-plugin`插件。使用方式是在核心实现模块的`pom`文件中添加如下方式进行禁用：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-deploy-plugin</artifactId>
    <configuration>
        <!-- 指定当前模块不deploy -->
        <skip>true</skip>
    </configuration>
</plugin>
```

## 声明了nexus仓库地址

具体声明如下：

```xml
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>Releases</name>
        <url>http://修改成需要的Releases的URL</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Snapshots</name>
        <url>http://修改成需要的Snapshots的URL</url>
    </snapshotRepository>
</distributionManagement>
```