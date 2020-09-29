<h3 align="center">MKDOC</h3>

<p align="center">
  <span>灵活可定制,多语言支持的API文档生成器</span>
  <br />
      <img src="https://goreportcard.com/badge/github.com/TheWinds/mkdoc">
      <img alt="GitHub" src="https://img.shields.io/github/license/thewinds/mkdoc">
  <br />

## TOC
* [介绍](#介绍)
* [特性](#特性)
* [开始使用](#开始使用)
    * [Playground](#Playground)
    * [CLI](#cli)
      * [安装](#安装)
      * [使用](#使用)
    * [DocServer](#docserver)
      * [安装&amp;部署](#安装部署)
      * [环境变量](#环境变量)
      * [配置文件](#配置文件)
    * [配置](#配置)
      * [配置格式](#配置格式)
      * [例子](#例子)
      * [详细说明](#详细说明)
    * [代码注解](#代码注解)
      * [gofunc 注解详细介绍](#gofunc-注解详细介绍)
* [例子](#例子-1)
* [原理](#原理)
* [鸣谢](#鸣谢)
* [END](#end)

## 介绍

mkdoc(make doc)是一款api文档生成器，相较于swagger等文档生成工具有使用简单易于拓展的特点。mkdoc致力于降低文档书写的负担,给开发者舒适的文档体验。

## 特性

- 良好的拓展性 🧩

  mkdoc将整个文档生成抽象为scanner、object loader、generator三个模块。如果你期望生成自己所需的文档格式(md、html等等)那么你仅需实现新的generator，而不需要考虑上层的工作机制。对于scanner和loader的拓展也是如此。

  同时提供了extension机制来实现拓展。

- 默认提供了简洁的注解语法 📖

  内建的 `gofunc` scanner 提供了一套简洁易用的文档注解语法。

- 配置简单易于部署 🔨 

  目前提供了cli和docker两种使用方式。可以使用cli生成文档后自行分发，也可以通过docker部署doc server配置git仓库后，由doc server接管文档的生成和展示。

  以上两种方式只需要一个额外的conf.yaml即可完成配置。

## 开始使用

### Playground

为了方便体验 mkdoc ,这里编译了WebAssembly版本（😎），点击下方链接即可在线体验。

[ 👉 在线体验 👈 ](http://mkdoc.thewinds.cn/)

### CLI

#### 安装

- 源码安装

  ```shell
  GO111MODULE=on go get github.com/thewinds/mkdoc/cmd/mkdoc
  ```

#### 使用

1. 初始化
    ```shell
    cd /path/to/your/projet
    ```
    ```shell
    mkdoc init
    ```

2. 参考[配置](#配置)章节进行配置
3. 参考[代码注解](#代码注解)进行文档注解
4. 生成文档
    ```shell
    mkdoc make
    ```

### DocServer

docserver提供了一个简单的文档服务，他在mkdoc之外增加了源码拉取和文档展示的功能，将docsify生成的网页直接展示出来，因此**必须启用docsify generator**。

除此之外docserver还支持多个项目，你可以在一个docserver上部署多个项目的文档服务。

1. 参考[配置](#配置)章节进行配置

2. 参考[代码注解](#代码注解)进行文档注解

#### 安装&部署

  - 拉取镜像
    ```shell
    docker pull thewinds/mkdoc-server
    ```
  - 启动（docker run）
    ```shell
    DOC_SERVER_GIT_USER_NAME=mkdoc \
    DOC_SERVER_GIT_PASSWORD=e257bf42 \
    DOC_SERVER_NOTIFY_TOKEN=b0b7f598d2e257bf42ef0b31dea14e9c \
    DOC_SERVER_WEB_USER_NAME=test \
    DOC_SERVER_WEB_PASSWORD=11111 \
    docker run \
        -v ${PWD}/conf.yaml:/mkdoc/conf.yaml \
        -p 20200:8080 \
        -e GIT_USER_NAME=${DOC_SERVER_GIT_USER_NAME} \
        -e GIT_PASSWORD=${DOC_SERVER_GIT_PASSWORD} \
        -e NOTIFY_TOKEN=${DOC_SERVER_NOTIFY_TOKEN} \
        -e WEB_USER_NAME=${DOC_SERVER_WEB_USER_NAME} \
        -e WEB_PASSWORD=${DOC_SERVER_WEB_PASSWORD} \
        thewinds/mkdoc-server
    ```
  - 启动（docker-compose）

    ```yaml
    # docker-compose.yml
    version: '2'

    services:
      mkdoc-server:
      image: thewinds/mkdoc-server
      ports:
        - 20200:8080
      environment:
        - GIT_USER_NAME=${DOC_SERVER_GIT_USER_NAME}
        - GIT_PASSWORD=${DOC_SERVER_GIT_PASSWORD}
        - NOTIFY_TOKEN=${DOC_SERVER_NOTIFY_TOKEN}
        - WEB_USER_NAME=${DOC_SERVER_WEB_USER_NAME}
        - WEB_PASSWORD=${DOC_SERVER_WEB_PASSWORD}
        - DEBUG=1
      volumes:
        - ./conf.yaml:/mkdoc/conf.yaml
      restart: always
    ```

  >  你将会看到以下输出:
  > ```
  >  2020/05/16 18:48:47 server docs:
  >  2020/05/16 18:48:47     index   =>      127.0.0.1:8080
  >  2020/05/16 18:48:47     project_0 =>      127.0.0.1:8080/project_0
  >  2020/05/16 18:48:47     project_1  =>      127.0.0.1:8080/project_1
  >  2020/05/16 18:48:47 notify url: 127.0.0.1:8080/notify
  >  ```

  容器在 `:8080` 端口提供文档web服务，触发文档重新生成的url为`:8080/notify?token=xxxxx`。

  #### 环境变量

| 名称| 描述 |
| --- | --- |
|GIT_USER_NAME|git repository 的用户名（私有仓库）|
|GIT_PASSWORD|git repository 的密码|
|NOTIFY_TOKEN|触发docserver重新生成文档的token|
|WEB_USER_NAME|basic auth用户名(非必须)|
|WEB_PASSWORD|basic auth密码(非必须)|
|DEBUG|DEBUG=1开启debug模式|

  > 如果 `WEB_USER_NAME` 不为空 basic auth 将会开启

  #### 配置文件
  配置文件必须命名为 `conf.yaml`

  配置文件包含多个章节，第一个章节是 `docserver`特有配置,其他章节的配置与 mkdoc CLI 的配置相同（[点击查看](#配置格式)）。

  - docserver专有配置

| 名称| 描述 |
| --- | --- |
|repo| repository to clone|
|branch| branch to clone|

  - projects 配置

| 名称| 描述 |
| --- | --- |
|id|path for doc page|

  例子:
  ```yaml
  repo: "https://github.com/TheWinds/mkdoc.git"
  branch: develop
  ---
  id: project_1
  name: mkdoc example1
  desc: this doc is auto generated by [mkdoc](https://github.com/TheWinds/mkdoc)
  api_base_url: "http://localhost:8080"
  mime:
    in:  form
    out: json
  scanner:
    - gofunc
  generator:
    - docsify
  args:
    enable_go_mod: true
    path: "./src"
  ---
  id: project_2
  name: mkdoc example2
  desc: this doc is auto generated by [mkdoc](https://github.com/TheWinds/mkdoc)
  api_base_url: "http://localhost:8080"
  mime:
    in:  form
    out: json
  inject:
    - name: "token"
      desc: "jwt token"
      default: "hfjdjhkklashjkfsd.hjkfsdajhkfdsj.jknsfdksf"
      scope: header
  scanner:
    - gofunc
    - docdef
  generator:
    - markdown
    - insomnia
    - docsify
  args:
    enable_go_mod: true
    path: "./src"
  ```

### 配置
  #### 配置格式

    配置文件为yaml格式，默认为conf.yaml
  - 字段说明

|字段|类型|说明|详情|
| ---- | ---- | ---- | ---- |
|name|string|项目名称|-|
|desc|string|项目描述|-|
|api_base_url|string|API域名前缀|-|
|inject|object|全局注入|[查看](#inject)|
|mime|object|全局api输入/输出媒体类型|[查看](#mime)|
|scanner|string array|启用文档扫描器列表|[查看](#scanner)|
|generator|string array|启用文档生成器列表|[查看](#generator)|
|args|obejct|全局参数，会被复制到每个scanner和generator|[查看](#args)|

  #### 例子

  ```yaml
  name: mkdoc example
  desc: this doc is auto generated by [mkdoc](https://github.com/TheWinds/mkdoc)
  api_base_url: "http://localhost:8080"
  mime:
    in:  form
    out: json
  inject:
    - name: "token"
      desc: "jwt token"
      default: "hfjdjhkklashjkfsd.hjkfsdajhkfdsj.jknsfdksf"
      scope: header
  scanner:
    - gofunc
    - docdef;path=./src/some/
  generator:
    - markdown
    - insomnia
    - docsify
  args:
    enable_go_mod: true
    path: "./src"
  ```
  #### 详细说明

  ##### inject
  inject选项用于配置一些通用的参数，例如你希望每个接口的header都带有一个token字段，那么你可以通过inject的方式来进行配置。这对于一些测试文件生成的generator来说是非常有用的，例如 `insomnia`。

  ##### mime
  mime选项用于配置全局api的输入输出的MIMEType。in为输入，out为输出。
  例如:
  ```yaml
  mime:
    in:  form # 输入为form表单
    out: json # 输出为json
  ```

  目前实现了对 form和json mime type的支持。

  ##### scanner
  scanner选项用于配置启用文档扫描器列表，您至少配置一个启用的文档扫描器。
  如果你希望向scanner传递参数，那么您应该在分号后用key=value的方式进行传递。格式为`scanner_name;param_name=param_value;param_name=param_value`多个参数之间用 `;` 分割，参数与scanner_name之间也用`;` 分割。

  ##### generator
  generator选项用于配置启用文档生成器列表，您至少配置一个启用的文档扫描器。
  如果你希望向generator传递参数，那么您应该在分号后用key=value的方式进行传递。格式为`generator_name;param_name=param_value;param_name=param_value`多个参数之间用 `;` 分割，参数与generator_name之间也用`;` 分割。

  ##### args
  args选项用于配置全局传递参数，这些参数会被复制到每个scanner和generator。但是他们的优先级是最低的，如果在scanner或generator也定义了同样的参数，那么全局的参数将被覆盖。

### 代码注解

  > 代码注解并不是必须的，例如自带的 `docdef` scanner 就不需要去定义注解，而是需要在*.doc.json中定义包含api列表和object列表的schema。

  注解的格式由scanner决定，mkdoc默认支持 `gofunc` scanner 下面介绍一下gofunc的注解语法。

  gofunc scanner 会从 go 源文件进行文档扫描，扫描的范围是 function declare 的comment，也就是func上面的注释。

  先上一个例子进行说明:
  ```go
    type CreateUserV2Req struct {
      // 用户名
      Name string `json:"name"`
      // 密码
      Password string `json:"pwd"`
      // 年龄
      Age int `json:"age"`
  }

  // @doc 创建用户
  // create user
  // some description
  // ...
  // @tag user,userv2
  // @path /api/v2/user @method post
  // @in  type CreateUserV2Req
  // @out type model.User
  func CreateUserV2() {
    // ...
  } 
  ```

  > 注解必须以`@doc`指令作为开头，才会被scanner识别。`@doc`指令后是api的名称。接着从`@doc`到下一个指令之间的若干行用来写对于该api的描述，描述并不是必须的。`@tag`指令指定了该api的tag，用于对api进行分组，多个tag直接用`,`分割。`@path`指令用于指定api相对于`api_base_url`的路径，`@method`指令用于指定method。`@in`和`@out`指令用于指定输入/输出的类型。

#### gofunc 注解详细介绍

|指令名称|说明|详情|
| ----  | ---- | ---- |
|@doc|API名称|-|
|@tag|API所属标签|-|
|@path|相对`api_base_url`的路径|-|
|@method|API所采用的方法(不一定是http method)|-|
|@query|表示 URL Query 参数，如果有多个可以重复该指令|-|
|@header|表示 HTTP Header 参数，如果有多个可以重复该指令|-|
|@in|启用文档生成器列表|[查看](#in)|
|@out|启用文档生成器列表|[查看](#out)|
|@disable|全局参数，会被复制到每个scanner和generator|[查看](#disable)|



##### @in

`@in` 指令用于指定输入参数,该指令有三种形式。
- 1.`@in type type_name`

    type_name为类型名称，格式为`包名.类型名`例如`model.User`。注意包名不需要为完整包名，loader会根据引用包名进行推导出完整包名。

    > type_name也可为go的基本类型（主要用于@out type）:
      ```
      string,bool,byte,
      int, int8, int16, int32, int64,
      uint, uint8, uint16, uint32, uint64,
      float, float32, float64,
      interface{}
      ```
    
    例子:
    ```go
    type SomeArg struct{
      // 名称
      Name string `json:"name"`
      // 年龄
      Age  int    `json:"age"`
    }
    // @doc A
    // ...
    // @in type SomeArg
    func A(){}

    // @doc B
    // ...
    // @in type string
    func B(){}
    ```
- 2.`@in fields {\n files_list \n}`

  files_list定义了一组参数，这样就不需要在代码中定义。

  例子:
  ```go
  // @doc A
  // ...
  // @in fields {
  //    name string 名称
  //    age  int    年龄
  // }
  func A(){}
  ```

- 3.`@in[mime_type] xx xx`

  mime_type 用于覆盖指定该接口入参的MIMEType，如果该接口的MIMEType与全局设置不一致可以通过这种方式进行单独设置。

  ```go
  // @doc A
  // ...
  // @in[form] fields {
  //    name string 名称
  //    age  int    年龄
  // }
  ```
##### @out

`@out` 指令用于指定输出参数与[@in指令](#@in)的用法一致。

##### @disable

`@disable` 指令用于屏蔽全局设置，比如想要禁用全局的header inject。则可以通过`@disable common_header`的方式进行禁用。



## 例子
参考[examples](https://github.com/TheWinds/mkdoc/tree/master/_examples)目录下的例子🌰
## 原理
TODO

## 鸣谢

特别感谢 [JetBrains](https://www.jetbrains.com/?from=ferry) 为本开源项目提供免费的 [IntelliJ GoLand](https://www.jetbrains.com/go/?from=ferry) 授权

<p>
 <a href="https://www.jetbrains.com/?from=ferry">
   <img height="200" src="https://www.fdevops.com/wp-content/uploads/2020/09/1599213857-jetbrains-variant-4.png">
 </a>
</p>

## END
感谢您关注此项目 : )，如果有好的想法欢迎 Issue or PR。
