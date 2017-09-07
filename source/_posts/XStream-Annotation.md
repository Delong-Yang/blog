---
title: XStream Annotation
date: 2017-05-24 11:01:46
tags: Java
---

最近接手一个项目的二次开发，使用XML作为数据传输格式， 利用XStream对XML进行序列化和反序列化。

# XStream常用的Annotations
<!-- more --> 
## `@XStreamAlias` 
XStream默认使用类名作为元素名， `@XStreamAlias`用来给xml中的元素起一个简短的别名 

```java
@XStreamAlias("query")
public class Query {
    @XStreamAlias("cobdate")
    private String cobDates;
    //ignore getters and setters
}
```

序列化结果：
```xml
<query>
  <cobdate>20170524</cobdate>
</query>
```

## `@XStreamAsAttribute` 
将类内成员作为父节点attribute输出，等同于xstream.useAttributeFor(Cat.class, "name")
```java
@XStreamAlias("query")
public class Query {
    @XStreamAsAttribute
    @XStreamAlias("cobdate")
    private String cobDates;
    //ignore getters and setters
}
```
序列化结果：
```xml
<query cobdate="20170524"/>
```
给成员添加`@XStreamAsAttribute`后，依然可以反序列化下面xml得到cobDate：
```xml
<query>
  <cobdate>20170524</cobdate>
</query>

Query{cobDates='20170524'}
```
去掉后， 反序列化下面xml， cobdate将得到null
```xml
<query cobdate="20170524"/>

Query{cobDates='null'}
```
## `@XStreamImplicit`
对于集合的注解， disable输出集合节点, 不加`@XStreamImplicit`输出结果为：
```java
@XStreamAlias("query")
public class Query {
    @XStreamAsAttribute
    @XStreamAlias("cobdate")
    private String cobDates;

    private List<String> aggregates;
    //ignore getters and setters
}
```
```xml
<query cobdate="20170524">
  <aggregates>
    <string>portfolio</string>
    <string>desk</string>
  </aggregates>
</query>
```
添加`@XStreamImplicit`结果变为， “itemFieldName”用于给集合内元素起别名
```java
@XStreamAlias("query")
public class Query {
    @XStreamAsAttribute
    @XStreamAlias("cobdate")
    private String cobDates;
    
    @XStreamImplicit(itemFieldName="aggregate")
    private List<String> aggregates;
    //ignore getters and setters
}
```
```xml
<query cobdate="20170524">
  <aggregate>portfolio</aggregate>
  <aggregate>desk</aggregate>
</query>
```
## `@XStreamOmitField` 
表明该属性不会被序列化到xml中。