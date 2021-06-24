## 条件转换

通常，工作流需要比不可变的状态序列更智能一点。 这时候你可能需要条件转换。 在本文中，您将了解实例输入数据、开关状态以及如何定义循环。

### 示例

```yml
id: multiposter
functions:
- id: httprequest
  image: vorteil/request:v2
states:
- id: ifelse
  type: switch
  conditions:
  - condition: '.names'
    transition: poster
- id: poster
  type: action
  action:
    function: httprequest
    input: '{
      "method": "POST",
      "url": "https://jsonplaceholder.typicode.com/posts",
      "body": {
        "name": .names[0]
      }
    }'
  transform: 'del(.names[0])'
  transition: ifelse
```

#### 输入

```json
{
  "names": [
    "Alan",
    "Jon",
    "Trent"
  ]
}
```

#### 输出

```json
{
  "names": [],
  "return": {
		"id": 101
	}
}
```

### 输入实例

可以使用可用作实例数据的输入数据调用工作流。 有一些 ifs-and-buts 适用于输入数据，但并没有那么复杂。



