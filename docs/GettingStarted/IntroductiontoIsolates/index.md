## 隔离介绍

如果工作流仅限于预定义的状态，它们就不会非常强大。 这就是为什么 Direktiv 可以运行“隔离”，它基本上是无服务器容器。 在本文中，您将了解 Action State 并了解 Isolates。

### 示例

```yml
id: httpget
functions:
- id: httprequest
  image: vorteil/request:v2
states:
- id: getter
  type: action
  action:
    function: httprequest
    input: '{
      "method": "GET",
      "url": "https://jsonplaceholder.typicode.com/todos/1",
    }'
```

此工作流将使用 https://hub.docker.com/r/vorteil/request 上的 Docker 容器执行 GET 请求并将结果返回到实例数据。

并非任何 Docker 容器都可以作为 Isolate 工作，但要使其兼容并不困难。 我们稍后再讨论。

运行此工作流。 暂时将工作流输入留空。 您应该会看到类似以下内容：

#### 输入
```json
{}
```

#### 输出

```json
{
  "return": {
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  }
}
```

“return”下的JSON结构就是GET请求返回的对象。


### 隔离介绍

隔离只是我们在运行无服务器容器时使用的一个奇特的术语。 Direktiv 从可用的 Docker 注册表中获取一个 Docker 容器： hub.docker.com 除非定义了自定义注册表（稍后会详细介绍）。 然后它将这个容器作为“函数”运行。 如果容器根据我们的 Isolate 要求处理输入和输出，它几乎可以做任何事情（稍后还会详细介绍我们的 Isolate 要求）。

#### 函数定义

```yml
functions:
- id: httprequest
  image: vorteil/request:v2
```

要使用隔离，它必须首先在工作流定义的顶部进行定义。 每个函数定义都需要一个在工作流定义中必须唯一的标识符，以及一个引用要使用的 Docker 容器的映像。

### 动作状态

```yml
- id: getter
  type: action
  action:
    function: httprequest
    input: '{
      "method": "GET",
      "url": "https://jsonplaceholder.typicode.com/todos/1",
    }'
```

像所有其他状态一样，动作状态需要一个 id 和类型字段来标识它。 但 Action State 的伟大之处在于它能够以“隔离”的形式运行用户制作的逻辑。

函数字段必须引用工作流定义中定义的函数之一。 在这个例子中，我们使用了 vorteil/request，它是一个简单的容器，它执行 HTTP 请求并返回结果。 我们使用输入字段中指定的 jq 命令来生成 Isolate 的输入。


一旦 Isolate 在 Action State 中完成其任务，结果将存储在“return”字段下的实例数据中。