# Pt Rule Engine

## 简介
`Pt Engine` 是一款安全、功能强大的通用规则引擎，旨在为企业和开发者提供高效、灵活且强大的规则决策解决方案。无论在金融、电商还是医疗等业务复杂的领域，都能发挥重要作用。

引擎由两部分组成：

- editor: 规则编辑器，提供规则的创建、编辑功能。在官网注册后即可使用[Pt Editor](https://www.pt-engine.com/editor/)

- executor: 规则执行器，负责规则的执行。是一个独立的二进制文件，在本地启动后通过`HTTP`请求交互。在`release`页面即可下载相应的执行器文件。

完整的使用文档可以参考[Pt Engine 文档中心](https://www.pt-engine.com/doc/index.html)

## 特点

### 简单易用
- **高效精准**: 快速准确处理业务规则，确保规则执行的准确性
- **简洁界面**: 提供简洁直观的用户操作界面，方便用户轻松上手，快速创建和管理规则。
- **可视化编辑**: 不需要专业编程技能，简单学习就能独立完成规则制定
- **执行轻量**: 独立的执行模块，`Pt Executor`无任何应用级依赖。
- **快速迭代**：独立更新业务规则，告别频繁重启业务服务。
- **适用广泛**：可用于金融交易、风控、电商策略、医疗等业务复杂的领域。


### 强大功能
- **自定义数据结构**：支持自定义响应数据结构，便于各类应用接口交互集成。
- **多样规则节点**：提供多种规则节点提供抽象，对复杂业务场景有出色的描述方式。
- **版本支持**：内置规则流版本支持，适配业务升级、迁移。
- **丰富内置函数**：支持丰富的数据运算，包括数值运算、字符串运算、时间函数等。
- **扩展节点类型（即将推出）**：如消息节点、API节点、DB节点等。

### 性能卓越
- **无 GC 特性**：可用于高频交易、游戏等性能敏感场景。
- **高效执行**：具备极快的执行效率，在高并发场景下有不俗的表现。
- **低内存占用**：极低的内存占用，适用于嵌入式、IoT、区块链等领域。
- **易于集成**：可编译成FFI库(未开放)，方便集成到各类应用中。

## Executor 使用

在规则流的`历史版本`页面有指定运行所需的最低`Pt Executor`版本信息。

下载合适的`pt-exec-srv`到运行服务器上启动

```shell
# 确保引擎有运行权限
chmod +x pt-exec-srv
# 默认监听0.0.0.0 8000端口，可通过`-p`参数指定其他端口
./pt-exec-srv
```

!>运行服务对`url`末尾的`/`有严格要求，不可省略。

文档以请求本地服务为示例

### 首页
检查引擎服务处于正常状态

```
curl -X GET 'http://localhost:8000/'
```

响应如看到如下字样说明引擎服务正常：

```
Welcome Platinum Engine!
```

## 上传规则流
规则流运行前需加载缓存到内存中。请求方式为：

```
curl -X POST -H "Content-Type: text/plain" --data-binary "@<file_path>" http://localhost:8000/flows/upload/
```

其中`<flow_path>`指用户所下载的规则流版本文件路径

响应如看到`Flow uploaded successfully`说明上传成功。

此外，上传的规则流会被放置于当前工作路径的`flows_cache`目录下，在下次启动`Pt Executor`时会自动加载


## 查看缓存规则流
```
curl -X GET 'http://localhost:8000/flows/'
```


## 删除规则流
```
curl -X POST 'http://localhost:8000/flows/delete/' \
--header 'Content-Type: application/json' \
--data '{
    "f_id": "RF008",
    "f_ver": "v20240731001"
}'
```

其中`f_ver`为可选参数，不写则删除`f_id`对应的全部规则流。

`flows_cache`目录下对应的规则流版本文件也会被删除



## 执行规则流
引擎运行服务的核心接口。请求`url`路径包含需要运行的`flow_id`和`version`。

```
curl -X POST 'http://localhost:8000/flows/<flow_id>/<version>/' \
--header 'Content-Type: application/json' \
--data '{
    "total_price": 400,
    "is_member": true
}'
```

> 如果只希望运行最新版本的规则流，则`<version>`部分使用`latest`即可。例如`http://localhost:8000/flows/RF008/latest/`

其中`data`分为为该规则流所需的字段信息。可通过编辑服务对应的规则流版本查看。

默认不对字段校验。如需额外做字段类型校验和存在校验，则需在`data`中加入参数`<STRICT>: true`。

如规则流正常运行结束，则请求响应结构为：

```
{
  code: 0,
  data: {
    final_price: 288
  }
}

```

其中`data`数据为对于`结束节点`中所配置的键值对信息。

`code`常见状态码有：

+ 0：正常
+ 1：请求参数异常
+ 3：规则流-版本 不存在
+ 40：规则流运行错误
