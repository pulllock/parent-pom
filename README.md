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
    <version>1.0.0-SNAPSHOT</version>
</parent>
```