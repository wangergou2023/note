





### key/value（键/值）对

格式

~~~json
key:value
~~~

**key**：必须是字符串。

**value**：可以是合法的 JSON 数据类型（字符串, 数字, 对象, 数组, 布尔值或 null）示例如下。

**key** 和 **value** 中使用冒号 **:** 分割。

每个 **key/value** 对使用逗号 **,** 分割。

* 字符串

  ~~~json
  {
      "name": "菜鸟教程"
  }
  ~~~

* 数字

  ~~~json
  {
      "age": 30
  }
  ~~~

* 逻辑值

  ~~~json
  {
      "age": true
  }
  ~~~

* null

  ~~~json
  {
      "age": null
  }
  ~~~

* 对象

  ~~~json
  {
      "age": {}
  }
  ~~~

* 对象数组

  ~~~json
  {
      "age": [
          {},
          {}
      ]
  }
  ~~~

  