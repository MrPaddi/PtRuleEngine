# Pt Rule Engine

[中文简介][zh-readme-url]

[zh-readme-url]: README-zh.md

## Introduction
`Pt Engine` is a secure and powerful universal rule engine, aiming to provide enterprises and developers with efficient, flexible and robust rule decision-making solutions. It can play an important role in complex business fields such as finance, e-commerce and healthcare.

The engine consists of two parts:

- Editor: The rule editor that provides the functions of creating and editing rules. It can be used after registering on the official website [Pt Editor](https://www.pt-engine.com/editor/).

- Executor: The rule executor, which is responsible for executing rules. It is an independent binary file and interacts through `HTTP` requests after being started locally. The corresponding executor files can be downloaded on the `release` page.

The complete usage documentation can be referred to [Pt Engine Doc](https://www.pt-engine.com/doc/index.html).

## Features

### Easy to Use
- **Efficient and Accurate**: It can quickly and accurately handle business rules and ensure the accuracy of rule execution.
- **Simple Interface**: It provides a simple and intuitive user operation interface, making it convenient for users to get started easily and quickly create and manage rules.
- **Visual Editing**: Professional programming skills are not required. Users can independently formulate rules after simple learning.
- **Lightweight Execution**: It has an independent execution module, and `Pt Executor` has no application-level dependencies.
- **Rapid Iteration**: Business rules can be updated independently, bidding farewell to frequent restarts of business services.
- **Widely Applicable**: It can be used in complex business fields such as financial transactions, risk control, e-commerce strategies and healthcare.

### Powerful Functions
- **Custom Data Structures**: It supports customizing response data structures, facilitating the integration of interactions with various application interfaces.
- **Diverse Rule Nodes**: It provides multiple rule nodes for abstraction and has an excellent way of describing complex business scenarios.
- **Version Support**: It has built-in support for rule flow versions, adapting to business upgrades and migrations.
- **Rich Built-in Functions**: It supports rich data operations, including numerical operations, string operations, time functions and so on.
- **Extended Node Types (Coming Soon)**: Such as message nodes, API nodes, DB nodes and so on.

### Excellent Performance
- **No GC Feature**: It can be used in performance-sensitive scenarios such as high-frequency trading and games.
- **Efficient Execution**: It has extremely fast execution efficiency and performs well in high-concurrency scenarios.
- **Low Memory Consumption**: It has extremely low memory usage and is suitable for fields such as embedded systems, IoT and blockchain.
- **Easy to Integrate**: It can be compiled into an FFI library (not yet open), facilitating integration into various applications. 

## Executor Usage

On the `versions` page of the rule flow, there is information about the minimum required `Pt Executor` version for specified operation.

Download the appropriate `pt-exec-srv` to the running server and start it.

```shell
# Ensure the engine has the right to run
chmod +x pt-exec-srv
# By default, it listens on port 8000 of 0.0.0.0. Other ports can be specified via the `-p` parameter.
./pt-exec-srv
```

!> The running service has strict requirements for the `/` at the end of the endpoint, and it cannot be omitted.

The documentation takes a request to a local service as an example.

### Home Page
Check if the engine service is in a normal state.

```
curl -X GET 'http://localhost:8000/'
```

If you see the following words in the response, it indicates that the engine service is normal:

```
Welcome Platinum Engine!
```

### Upload Rule Flow
Before running a rule flow, it needs to be loaded into the cache in memory. The request method is:

```
curl -X POST -H "Content-Type: text/plain" --data-binary "@<file_path>" http://localhost:8000/flows/upload/
```

where `<flow_path>` refers to the path of the rule flow version file downloaded by the user.

If you see `Flow uploaded successfully` in the response, it indicates that the upload is successful.

In addition, the uploaded rule flow will be placed in the `flows_cache` directory of the current working path and will be automatically loaded when starting `Pt Executor` next time.

### View Cached Rule Flows
```
curl -X GET 'http://localhost:8000/flows/'
```

### Delete Rule Flows
```
curl -X POST 'http://localhost:8000/flows/delete/' \
--header 'Content-Type: application/json' \
--data '{
    "f_id": "RF008",
    "f_ver": "v20240731001"
}'
```

where `f_ver` is an optional parameter. If not specified, all rule flows corresponding to `f_id` will be deleted.

The corresponding rule flow version files in the `flows_cache` directory will also be deleted.

### Execute Rule Flows
This is the core interface of the engine running service. The requested `url` path contains the `flow_id` and `version` that need to be run.

```
curl -X POST 'http://localhost:8000/flows/<flow_id>/<version>/' \
--header 'Content-Type: application/json' \
--data '{
    "total_price": 400,
    "is_member": true
}'
```

> If you only want to run the latest version of the rule flow, use `latest` for the `<version>` section. For example` http://localhost:8000/flows/RF008/latest/ `

where `data` is the field information required for this rule flow. It can be viewed by editing the corresponding rule flow version of the service.

By default, there is no field verification. You can obtain request field information by downloading version page information

If the rule flow ends normally, the request response structure is:

```
{
  code: 0,
  data: {
    final_price: 288
  }
}
```

where `data` is the key-value pair information configured in the `end node`.

Common `code` status codes are:

+ 0: Normal
+ 1: Request parameter abnormal
+ 3: Rule flow - version does not exist
+ 40: Rule flow running error