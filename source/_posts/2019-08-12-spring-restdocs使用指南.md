---
title: Spring Restdocs使用指南
date: 2019-08-12 09:59:34
tags:
- java
- spring
- restdocs
categories:
- java
- spring
- restdocs
---
# 前言
本文旨在介绍Spring restdocs的相关用法，在这之前需要对restful有比较全面的了解，哪怕仅仅只是概念上的，所以接下来在介绍restdocs的使用方法前，得先简单的概述下restful相关的知识。
<!--more-->

# Rest架构介绍
## 什么是Rest？
> 1. REST 是 Representational state transfer 的缩写，翻译为表达性状态转换。
> 2. REST 是一种架构风格，它包含了一个分布式超文本系统中对于组件、连接器和数据的约束。
> 3. REST 是作为互联网自身架构的抽象而出现的，其关键在于所定义的架构上的各种约束。

只有满足这些约束，才能称之为符合 REST 架构风格

## 为什么要使用Rest
从实用角度讲，现在的互联网应用通常都是一个数据服务对应多个客户端，这些客户端可能分布于其它各个平台，Restful以http协议调用的方式提供一套统一的接口为各个客户端提供服务，为各个平台提供相同的使用体验，同时restful带有自描述的特征，根据action和method我们就可以知道接口需要做什么事情。

从技术角度讲restful可以让前端知道的更少，减少硬编码的数量，做到更加的智能和自适应。

## Rest的成熟模型
REST 成熟度模型把 REST 服务按照成熟度划分成 4 个层次（成熟度由低到高）：
> **第一层次（Level 0）的 Web 服务只是使用 HTTP 作为传输方式，实际上只是远程方法调用（RPC）的一种具体形式。SOAP 和 XML-RPC 都属于此类**
> **第二层次（Level 1）的 Web 服务引入了资源的概念。每个资源有对应的标识符和表达**
> **第三层次（Level 2）的 Web 服务使用不同的 HTTP 方法来进行不同的操作，并且使用 HTTP 状态码来表示不同的结果，如：HTTP GET 方法来获取资源，HTTP DELETE 方法来删除资源**
> **第四层次（Level 3）的 Web 服务使用 HATEOAS。在资源的表达中包含了链接信息，客户端可根据链接来发现可执行的动作。**

## 啥是HATEOAS？
[Spring HATEOAS](https://spring.io/projects/spring-hateoas) 是一个用于支持实现超文本驱动的 REST Web 服务的开发库。是 HATEOAS 的实现，HATEOAS背后的思想就是响应中包含指向其它资源的链接，客户端可以利用这些链接和服务器交互。
{% asset_image 1.png %}
**常用的非HATEOAS请求响应：**
``` code
GET /posts/1 HTTP/1.1
Connection: keep-alive
Host: example.ycb.com
{
    "id" : 1,
    "body" : "My first blog post",
    "postdate" : "2015-05-30T21:41:12.650Z"
￼}
```

**HATEOAS的响应例子：**
``` code
"id" : 1,
"body" : "My first blog post",
"postdate" : "2015-05-30T21:41:12.650Z",
"links" : [
    {
        "rel" : "self",
        "href" : http://example.ycb.com/posts/1,
        "method" : "GET"
    }
] 
```
上面的例子中，每一个在links中的link都包含了三部分：

> href：用户可以用来检索资源或者改变应用状态的URI
> rel：描述href指向的资源和现有资源的关系
> method：和此URI需要的http方法

**HATEOAS（Hypermedia as the engine of application state）是 REST 架构风格中最复杂的约束，也是构建成熟 REST 服务的核心，它的重要性在于打破了客户端和服务器之间严格的契约，使得客户端可以更加智能和自适应，而 REST 服务本身的演化和更新也变得更加容易。**

# Rest docs介绍
Spring REST Docs的目的是帮助您为RESTful服务生成准确且可读的文档。

编写高质量的文档很困难。缓解这种困难的唯一方法是使用非常适合的工具。为此，Spring REST Docs 默认使用 [Asciidoctor](https://asciidoctor.org/docs/asciidoc-syntax-quick-reference)。Asciidoctor处理纯文本并生成HTML，样式和布局以满足您的需求。如果您愿意，还可以将Spring REST Docs配置为使用Markdown。

Spring REST Docs使用由Spring MVC的测试框架，Spring WebFlux WebTestClient或 REST Assured 3编写的测试产生的片段 。这种以测试为导向的方法有助于保证服务文档的准确性。如果代码段不正确，则生成代码段的测试将失败。

记录RESTful服务主要是描述其资源。每个资源描述的两个关键部分是它消耗的HTTP请求的详细信息以及它产生的HTTP响应。Spring REST Docs允许您使用这些资源以及HTTP请求和响应，从而使您的文档免受服务实现的内部细节的影响。这种分离可以帮助您记录服务的API而不是其实现。关于API的变动我们不需要对文档做出任何改动。

## 官网地址
[https://spring.io/projects/spring-restdocs](https://spring.io/projects/spring-restdocs "https://spring.io/projects/spring-restdocs")

## Rest docs VS Swagger
Restdocs和swagger都是当今很主流的技术文档自动生成工具，他们在某些技术层面有相似的成本，他们也同样的优秀，这里我无法比较到底谁更好，但是可以谈谈我在选择文档工具时几个考虑的点，提供大家参考：
1. **重所周知swagger在java里面生成文档，是需要在controller里面添加注释的，如果接口改动足够频繁，修改的范围足够广，很难保证开发者在此过程中不会遗漏，而且分散在各个controller的注解修改本身也会是一件头疼的事情，而rest docs则是以测试用例为导向，在保证接口的可用及正确性的前提下，同时也规范了开发流程，api接口的代码改动将会通过测试代码自动修正，可谓是一举两得。**

2. **Rest docs会将每个请求的各个部分生成不同的代码片段，同时使用asciidoc进行文档编写，它可以使得文档的生成有更多的选择，更加灵活，而且专业性及可读性也会更胜一筹。**
3. **最为关键的一点在于，通过测试用例的方式生成文档使得在分布式系统的场景下各个应用的接口文档生成在统一应用下成为可能，它纯粹以模拟用户请求的方式来解析api的各个环节，这样就避免的文档工具与代码本身的紧耦合，我们可以方便的决定我们的文档应该放置于什么地方进行集中编写和管理。**
4. **restdocs性能更好，它的特性使得文档生成与每个服务的代码没有耦合，可以方便我们在不同的场景下处理文档的生成及访问规则，安全性也更好。**

以上使我在选择文档工具着重考量的几个点，如果还有其它方面的问题欢迎大家补充。

# Rest docs的使用
本文的使用方法以spring cloud项目为例，可以为大家在分布式系统架构下面使用文档生成工具提供一些参考，其中讲解及演示的部分仅限于项目所需要的部分，完整的使用指导请参考官方文档。

## 添加依赖
pom.xml:
``` bash
...

<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <version>${restdocs.version}</version>
    <scope>test</scope>
</dependency>

...

<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>1.5.3</version>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process-asciidoc</goal>
            </goals>
            <configuration>
                <backend>html</backend>
                <doctype>book</doctype>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-asciidoctor</artifactId>
            <version>${restdocs.version}</version>
        </dependency>
    </dependencies>
</plugin>

...
```
## 编写测试用例
测试用例的编码方案在官方指南里分为多种方案，我们这里采用Junit4 + mockMvc的组合方案，不同的方案用法及限制会有不同，这点需要注意。

示例：
``` code
@RunWith(SpringRunner.class)
@SpringBootTest(classes = GatewayApplication.class)
public class RestDocTest {
    @Rule
    public JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation();

    private MockMvc mockMvc;
    @Autowired
    private WebApplicationContext context;

    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                .apply(
                        documentationConfiguration(this.restDocumentation)
                        .operationPreprocessors()
                        .withResponseDefaults(prettyPrint())
                )
                .build();
    }

    @Test
    public void testAuthenticate() throws Exception {...}
}
```

## 生成api文档片段
项目的结构目录：
{% asset_image 2.png %}

**这个时候我们如果执行了刚才编写的测试用例，reset docs默认会在target的generated-snippets下面生成对应的文档片段。**

## 编写api文档
文档模板默认是在src下面的asciidoc目录下：
``` code
...

.请求参数描述
include::{snippets}/auth/request-parameters.adoc[]

.request
include::{snippets}/auth/http-request.adoc[]

.响应数据结构描述
include::{snippets}/auth/response-fields.adoc[]

.response
include::{snippets}/auth/http-response.adoc[]

...
```
通过如上的方式可以讲测试阶段生成的代码片段引入到我们自己编写的文档中，我们可以进行灵活的配置及文档的定制。

## 生成完整的文档
项目发布的时候如果执行了package命令，会根据我们自定义的文档模板生成对应的文档，生成后的文档默认是在target下面的generated-docs目录下。

至此，我们的文档就生成完毕，我们根据自己的实际情况进行部署即可。