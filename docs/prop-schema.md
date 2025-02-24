# propsSchema 配置

### 概述

- `propsSchema` 是 FormRender 的必填 props，用于描述表单的基本信息、结构和校验。
- `propsSchema`遵从和使用了`JSON Schema`的约定规范。已经接入`JSON Schema`标准的团队可以几乎无缝接入`form-render`。`JSON Schema`是一个约定了可用的结构和字段的特殊 json，作为国际标准，主要应用于校验 JSON 数据
- 在少数写法抉择上`propsSchema`并未完全遵守`JSON Schema`，目前主要两个区别：

  - 引入了新类型`range`
  - 使用字段 `enumNames`，用于描述下拉单选的选项文案（enumNames 曾经是 JSON Schema 的 draft 提案，但最后被否绝了）
  - 这是权衡各类用户使用便利性的结果。毕竟`JSON Schema`是为了校验数据而生的，与表单的场景的侧重点是不尽相同的。当然`propsSchema`规范坚守的原则是对于使用`JSON Schema`标准的用户做到 schema 不改一字快速接入

- 通过 `JSON Schema` 里的字段可以描述表单的标题、描述、类型、必须项、自定义正则校验等信息。想深入了解的同学，<a href="https://json-schema.org/understanding-json-schema/" target="_blank">Understanding JSON Schema</a>是笔者认为最好的学习文档，同时也可去 <a href="https://alibaba.github.io/form-render/docs/demo/index.html" target="_blank">FormRender Playground</a> 折腾
- 虽然这里我们只以 json 格式为例，但 javascript object 作为入参完全可以

一个基础的 propsSchema 如下：

```json
{
  "type": "object",
  "properties": {
    "jobNumber": {
      "title": "数字",
      "type": "number"
    }
  }
}
```

描述了一个 object 结构，其第一个属性为数字类型。最外层约定为 object 结构，所有 propsSchema 都需要如是写。

### 通用参数

对于每一个表单控件，我们都会使用如下的 schema 描述

```json
{
  "title": "数字",
  "type": "number"
}
```

- `title`：表单的标题信息，作为 label 展示，注意 title 为""时占位，title 不写时不占位
- `description`：表单的描述信息，常将填写注意点放入此参数
- `type`：表单的类型，支持 string、number、boolean、array、object、range
- `format`：用来描述输入框的格式，支持 image、dateTime、date、time
- `pattern`：自定义正则校验，用于校验 string 或 number 数据是否合格，详细使用可见 <a href="https://alibaba.github.io/form-render/#/docs/pattern" target="_blank">pattern 自定义正则校验</a>
- `message` 校验提示自定义文案，与 pattern 共同使用
- `default` 默认值，对象类型不能使用 default，其他类型包括 array 都可以使用 default：

```json
"list": {
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "x": {
        "type": "string",
      }
    }
  },
  "default": [{ "x": "a" }, { "x": "b" }]
}
```

#### String

string 类对应的控件非常多, 使用 `format` 字段指定使用组件：

```json
// 默认 input
"input": {
  "title": "简单输入框",
  "type": "string",
}
// textarea
"textarea": {
  "title": "简单文本编辑框",
  "type": "string",
  "format": "textarea"
}
// 颜色组件
"color": {
  "title": "颜色选择",
  "type": "string",
  "format": "color"
}
// 日期组件
"date": {
  "title": "日期选择",
  "type": "string",
  "format": "date" // "dateTime"
}
// 时间组件
"time": {
  "title": "时间选择",
  "type": "string",
  "format": "time"
}
// 图片链接组件
"image": {
  "title": "图片展示",
  "type": "string",
  "format": "image"
}
```

注意的字段：

- 输入框：input、textarea

  - `minLength`：字符串最小长度
  - `maxLength`：字符串最大长度

- 单选（类型也可能是 number）

  - `enum` 选项值
  - `enumNames` 选项的文案

```json
{
  "title": "单选",
  "type": "string",
  "enum": ["hz", "wh", "gy"],
  "enumNames": ["杭州", "武汉", "贵阳"]
}
```

#### Number

- `min`：数字最小值
- `max`：数字最大值
- `step`：允许递增的区间

#### Object

- `properties`：描述 object 的结构，必要属性
- `required`：描述对象下哪些项必填，非必要属性。为数组结构，每项是对应必填组件的 name

```json
{
  "title": "用户信息",
  "type": "object",
  "properties": {
    "tickets": {
      "title": "门票数",
      "type": "number"
    }
  },
  "required": ["tickets"]
}
```

#### Array

`Array`的数据结构可能是：列表 & 多选框

- `items`：用于描述 Array 中每个 item 的结构、类型

- 列表：
  - `minItems`：最少数组项为几项
  - `maxItems`：最多数组项为几项
  - `uniqueItems`：用于判断数组的元素是否有重复。`uniqueItems` 的值支持 boolean 和 string 两种类型

```js
// 1. 判断列表元素是否有重复
"uniqueItems": true
// 校验结果： 不通过。存在重复元素
[
  { "id": 1, "type": "topic" },
  { "id": 1, "type": "topic" }
]
// 2. 判断列表元素的某个特定属性是否有重复，例如 id
"uniqueItems": "id"
// 校验结果：不通过。存在重复的 id 值
[
  { "id": 3, "type": "vote" },
  { "id": 3, "type": "topic" }
]
```

```json
{
  "title": "对象数组",
  "type": "array",
  "minItems": 1,
  "maxItems": 3,
  "uniqueItems": true,
  "items": {
    "type": "object",
    "properties": {
      "tickets": {
        "title": "门票数",
        "type": "number"
      }
    }
  }
}
```

- 多选框

  - `enum`：参考单选
  - `enumNames`：参考单选, 只写 enum 不写 enumNames 时，会默认使用 enum 作为 enumNames

```json
{
  "title": "多选",
  "type": "array",
  "items": {
    "type": "string"
  },
  "enum": ["hz", "wh", "gy"],
  "enumNames": ["杭州", "武汉", "贵阳"]
}
```

#### Range

长度为 2 的 array，目前支持的组件为时间范围

```json
{
  "title": "日期范围",
  "type": "range",
  "format": "dateTime",
  "ui:options": {
    "placeholder": ["开始", "结束"]
  }
}
```

### 一个很全的结构

```json
{
  "type": "object",
  "properties": {
    "stringDemo": {
      "title": "字符串",
      "description": "英文或数字组合",
      "type": "string",
      "pattern": "^[A-Za-z0-9]+$",
      "message": {
        "pattern": "请输入正确格式"
      }
    },
    "imgDemo": {
      "title": "图片",
      "type": "string",
      "format": "image",
      "default": "'https://img.alicdn.com/tfs/TB1P8p2uQyWBuNjy0FpXXassXXa-750-1334.png'"
    },
    "disabledDemo": {
      "title": "不可用",
      "type": "string",
      "default": "我是一个被 disabled 的值"
    },
    "enumDemo": {
      "title": "枚举",
      "enum": ["A", "B"],
      "enumNames": ["养成", "动作"]
    },
    "dateDemo": {
      "title": "时间",
      "format": "dateTime",
      "type": "string"
    },
    "objDemo": {
      "title": "单个对象",
      "description": "这是一个对象类型",
      "type": "object",
      "properties": {
        "isLike": {
          "title": "单选项",
          "type": "boolean",
          "default": true
        },
        "background": {
          "title": "颜色选择",
          "description": "特殊面板",
          "format": "color",
          "type": "string"
        }
      }
    },
    "arrDemo": {
      "title": "对象数组",
      "description": "对象数组嵌套功能",
      "type": "array",
      "minItems": 1,
      "maxItems": 3,
      "items": {
        "type": "object",
        "properties": {
          "name": {
            "title": "字符名称",
            "description": "string类型",
            "type": "string",
            "pattern": "^[A-Za-z0-9]+$"
          },
          "num": {
            "title": "数字参数",
            "description": "number类型",
            "type": "number"
          }
        }
      }
    }
  },
  "required": ["stringDemo", "dateDemo"]
}
```
