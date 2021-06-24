## Transitions

将许多不同的操作串在一起的能力是工作流的基本组成部分。 在本文中，您将了解转换、变换和 jq。

### 示例

```yml
id: transitioner
states:
- id: a
  type: noop
  transform: '{
    "number": 2,
    "objects": [{
      "k1": "v1"
    }]
  }'
  transition: b
- id: b
  type: noop
  transform: '.multiplier = 10'
  transition: c
- id: c
  type: noop
  transform: '.result = .multiplier * .number | del(.multiplier, .number)'
  transition: d
- id: d
  type: noop
  transform: '.objects[0]'
```

#### 输入

```
{}
```

#### 输出

```json
{
  "k1": "v1"
}
```

#### 日志
```json
[10:10:30] Beginning workflow triggered by API.
[10:10:30] Running state logic -- a:1 (noop)
[10:10:30] State data:
{}
[10:10:30] Transforming state data.
[10:10:30] Transitioning to next state: b (1).
[10:10:30] Running state logic -- b:2 (noop)
[10:10:30] State data:
{
  "number": 2,
  "objects": [
    {
      "k1": "v1"
    }
  ]
}
[10:10:30] Transforming state data.
[10:10:30] Transitioning to next state: c (2).
[10:10:30] Running state logic -- c:3 (noop)
[10:10:30] State data:
{
  "multiplier": 10,
  "number": 2,
  "objects": [
    {
      "k1": "v1"
    }
  ]
}
[10:10:30] Transforming state data.
[10:10:30] Transitioning to next state: d (3).
[10:10:30] Running state logic -- d:4 (noop)
[10:10:30] State data:
{
  "objects": [
    {
      "k1": "v1"
    }
  ],
  "result": 20
}
[10:10:30] Transforming state data.
[10:10:30] Workflow completed.
```

### 过渡

一个工作流定义中可以定义多个状态。 每个都在状态字段下开始，并且可以通过在状态列表中查找表示新对象的破折号来区分多个状态。 在演示中有四种不同的状态：

#### 状态 'a'

```yml
- id: a
  type: noop
  transform: '{
    "number": 2,
    "objects": [{
      "k1": "v1"
    }]
  }'
  transition: b
```

#### 状态 'b'

```yml
- id: b
  type: noop
  transform: '.multiplier = 10'
  transition: c
```

#### 状态 'c'
```yml
- id: c
  type: noop
  transform: '.result = .multiplier * .number | del(.multiplier, .number)'
  transition: d
```

#### 状态 'd'

```yml
- id: d
  type: noop
  transform: '.objects[0]'
```

我们这里只有 Noop 状态，但大多数状态类型可以选择有一个转换字段，引用工作流定义中状态的标识符。 在状态完成运行后，Direktiv 使用此字段来确定实例是否已到达终点。 如果定义了到另一个状态的转换，则实例将继续到该状态。

在这个演示中，四个 Noop 状态以 a → b → c → d 的简单顺序定义。 每个状态的实例数据都继承自其前身，这就是使用 Transforms 会有所帮助的原因。

### Transforms & JQ

每个工作流实例总是有一个叫做“实例数据”的东西，它是一个用于传递数据的 JSON 对象。 几乎在工作流定义中可能发生转换的任何地方都可能发生转换，从而允许作者过滤、丰富或以其他方式修改实例数据。

转换字段可以包含一个有效的 jq 命令，该命令将应用于现有实例数据以生成一个新的 JSON 对象，该对象将完全替换它。 请注意，只有 JSON 对象才会被视为此 jq 命令的有效输出：jq 能够输出原语和数组，但这些不是转换可接受的输出。

因为 Noop 状态在应用其转换和转换之前记录其实例数据，所以我们可以在整个演示中跟踪这些转换的结果。

#### 输入

```json
{}
```

#### 第一次转换

第一个转换定义了一个全新的 JSON 对象。

#### 指令

```json
transform: '{
    "number": 2,
    "objects": [{
      "k1": "v1"
    }]
  }'
```

#### 结果实例数据

```json
{
  "number": 2,
  "objects": [
    {
      "k1": "v1"
    }
  ]
}
```

#### 第二次转换

```
第二个转换通过向现有实例数据添加新字段来丰富现有实例数据。
```

#### 指令

```
transform: '.multiplier = 10'
```

#### 结果实例数据

```json
{
  "multiplier": 10,
  "number": 2,
  "objects": [
    {
      "k1": "v1"
    }
  ]
}
```

#### 第三次转换

第三个转换将两个字段相乘以产生一个新字段，然后将结果通过管道传输到另一个删除两个字段的命令中。

#### 指令

```yml
transform: '.result = .multiplier * .number | del(.multiplier, .number)'
```

#### 结果示例数据

```json
{
  "objects": [
    {
      "k1": "v1"
    }
  ],
  "result": 20
}
```

#### 第四次转换

第四个转换选择一个嵌套在实例数据中的子对象，并将其变成新的实例数据。

#### 指令
```yml
transform: '.objects[0]'
```

#### 结果示例数据

```json
{
  "k1": "v1"
}
```