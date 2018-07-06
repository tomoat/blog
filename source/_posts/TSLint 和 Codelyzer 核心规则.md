```
# TSLint 和 Codelyzer 核心规则

[TOC]

## TSLint 核心规则

see [tslint core rules](https://palantir.github.io/tslint/rules/)

> arrow-return-shorthand

(建议)将 `() => { return x; }` 简写成 `() => x`。

> callable-types

(建议)(TS Only)只有一个函数签名的接口或者字面类型可以写成函数类型。

> class-name

(强制)类和接口名使用 Pascal 命名方式。

​```js
class Shape {}
interface SearchParam {}
​```

> comment-format

(强制)单行注释使用统一的格式规则。

可选参数如下：

- `check-space`：所有单行注释必须以一个空格开头，即 `// comment` 形式
- `check-lowercase`：注释第一个字母必须为小写字母(英文适用)
- `check-uppercase`：注释第一个字母必须为大写字母(英文适用)

一个配置例子如下：

​```shell
"comment-format": [true, "check-lowercase", {"ignore-words": ["TODO", "HACK"]}]
​```

> curly

(强制)不能省略 `if/for/do/while` 等语句的大括号。

> eofline

(建议)每个文件必须以一个新行(newline)结尾。

> forin

(建议)如果使用 `for...in` 语法，必须要使用 `if` 来过滤非自身属性。

​```js
for (let key in someObject) {
    if (someObject.hasOwnProperty(key)) {
        // code here
    }
}
​```

> import-blacklist

(建议)禁止直接使用 `import` 或者 `require` 整个模块；使用时，只能导入该模块的子模块。

例如，rxjs 代码库特别大，很多运算符都用不大，因此可以针对 rxjs 库编写规则如下：

​```js
"import-blacklist": [
    true,
    "rxjs"
]
​```

> import-spacing

(建议)保证 import 语句大括号旁边有空格，也即：

​```js
import { Component, Pipe } from '@angular/core';
​```

> indent

(强制)缩进使用统一的 tabs 或 spaces。

例如设置4个空格的设置如下：

​```shell
"indent": [true, "spaces", 4]
​```

> interface-over-type-literal

(TS Only)定义接口时，优先采用接口声明方式(`interface T { ... }`)，而不是类型字面量形式(`type T = { ... }`)。

> label-position

(强制)只允许 label 出现在合理的位置，例如 (`do/while/for/switch`) 语句。

> max-line-length

限定一行的最大长度，例如设置一行不能超过 140 个字符，设置如下：

​```js
"max-line-length": [
    true,
    140
]
​```

> member-access

(TS Only) 对类的成员需要显示声明可访问性。

可选参数如下：

- "no-public"：禁止指定 public，因为这是默认的
- "check-accessor"：强制 `get/set` 访问器需要显示声明
- "check-constructor"：构造函数需要显示声明

> member-ordering

强制成员顺序。

例如静态变量需要先于实例变量，变量先于函数，可以设置如下：

​```js
"member-ordering": [
    true,
    "static-before-instance",
    "variables-before-functions"
]
​```

> no-arg

禁止使用 `arguments.callee`，因为这让编译器无法优化代码，会导致潜在的性能问题。

> no-bitwise

禁止使用位运算符（除了 `&` 和 `|` 外）。

> no-console

禁止使用 console 语句（生产环境代码）。

> no-construct

禁止直接使用 `String`、`Number` 和 `Boolean` 构造函数。

可以使用 `Number(foo)` 形式，但是不能使用 `new Number(foo)` 形式。

> no-debugger

禁止出现 debugger 语句（生产环境代码）。

> no-duplicate-super

如果构造函数中 `super` 调用出现了多次，给出警告。

> no-empty

不允许出现空的语句块。

> no-empty-interface

(TS Only) 禁止空的接口。

> no-eval

禁止使用 `eval` 语句。

> no-inferrable-types

(TS Only) 对于简单的变量类型不需要显示指定类型（例如 number, string, boolean）。

​```js
class A {
    // recommend
    age = 18;

    // not recommend, because it can be easily inferred by the compiler
    name: string = 'xiaoming';
}
​```

可选参数如下：

- ignore-params：函数参数使用可推断类型
- ignore-properties：类的属性使用可推断类型

一个例子如下：

​```js
"no-inferrable-types": [true, "ignore-params", "ignore-properties"]
​```

> no-misused-new

(TS Only) 如果显示给接口定义构造函数或者给类定义 `new`，给出警告。

> no-non-null-assertion

(TS Only)禁止非空推断。

> no-shadowed-variable

禁止覆盖变量声明。

> no-string-literal

禁止非必要的字符串字面量访问，例如允许 `obj[prop-erty]`，而不允许 `obj['propety']`。

> no-string-throw

Flags throwing plain strings or concatenations of strings because only Errors produce proper stack traces.

> no-switch-case-fall-through

禁止 case 语句 fall through（也就是没有用 break 语句）。

例如:

​```js
switch (foo) {
    case 1:
        someFunc(foo);
    case 2:
        someOtherFunc(foo);
}
​```

然而这种是合理的（存在多个连续的 case 语句）：

​```js
switch(foo) {
    case 1:
        someFunc(foo);
        /* falls through */
    case 2:
    case 3:
        someOtherFunc(foo);
}
​```

> no-trailing-whitespace

不允许尾部空格。

> no-unnecessary-initializer

禁止 `var/let` 语句或者解构语句初始化成 `undefined`。

> no-unused-expression

禁止未使用的表达式语句（没有被赋值给变量或者函数调用）。

> no-use-before-declare

变量必须先声明后使用。

> no-var-keyword

不允许使用 var 关键字。

> object-literal-sort-keys

对象中的 key 必须按照字母表顺序排列，在避免 merge 冲突时有用。

> one-line

一些特定的关键字需要出现在同一行，例如 `catch`、`finally`、`else`、`open-brace`

​```js
// 推荐
try {

} catch {

} finally {

}

// 不推荐
if (foo) {

}
else {

}
​```

> prefer-const

优先使用 const。

> quotemark

引号设置，例如设置使用单引号：

​```js
"quotemark": [true, "single"]
​```

> radix

`parseInt` 必须提供进制参数。

> semicolon

分号设置，例如设置必须要分号：

​```js
"semicolon": ["always"]
​```

> triple-equals

比较需要使用 `===` 和 `!==`。

可选参数如下：

- "allow-null-check" 当比较 null 时可以使用 == 和 !=
- "allow-undefined-check" 当比较 undefined 时可以使用 == 和 !=

> typedef-whitespace

(TS Only) 类型定义时需不需要空格。

​```js
"typedef-whitespace": [
    true,
    {
        "call-signature": "nospace",
        "index-signature": "nospace",
        "parameter": "nospace",
        "property-declaration": "nospace",
        "variable-declaration": "nospace"
    }
]
​```

> typeof-compare

保证 `typeof` 和正确的字符串类型来比较。

> unified-signatures

(TS Only) 如果两个函数可以通过 optional 和 rest 参数合并成一个，给出警告。

> variable-name

检测变量名称可能出现的各种错误。

> whitespace

强制空格样式规范。

## Codelyzer 核心规则

see [codelyzer core rules](http://codelyzer.com/rules/)

> [directive-selector](http://codelyzer.com/rules/directive-selector/)

指令选择器允许命名方式。

​```js
"directive-selector": [
    true,
    "attribute",
    "app",
    "camelCase"
]
​```

> [component-selector](http://codelyzer.com/rules/component-selector/)

组件选择器允许命名方式。

​```js
"component-selector": [
    true,
    "element",
    "app",
    "kebab-case"
]
​```

> [use-input-property-decorator](http://codelyzer.com/rules/use-input-property-decorator/)

使用 `@Input()` 装饰器，而不是组件或者指令元数据中的 `inputs` 属性。

> [use-output-property-decorator](http://codelyzer.com/rules/use-output-property-decorator/)

使用 `@Output()` 装饰器，而不是组件或者指令元数据中的 `outputs` 属性。

> [use-host-property-decorator](http://codelyzer.com/rules/use-host-property-decorator/)

使用 `@HostProperty` 装饰器而不是组件或者指令元数据中的 `host` 属性。

> [no-input-rename](http://codelyzer.com/rules/no-input-rename/)

是否允许重命名 Input 变量。

> [no-output-rename](http://codelyzer.com/rules/no-output-rename/)

是否允许重命名 Output 变量。

> [use-life-cycle-interface](http://codelyzer.com/rules/use-life-cycle-interface/)

如果使用生命周期函数，必须在组件中实现该接口。

> [use-pipe-transform-interface](http://codelyzer.com/rules/use-pipe-transform-interface/)

管道必须实现 `PipeTransform` 接口。

> [component-class-suffix](http://codelyzer.com/rules/component-class-suffix/)

组件名必须添加 `Component` 后缀（后缀可自定义）。

> [directive-class-suffix](http://codelyzer.com/rules/directive-class-suffix/)

指令名必须添加 `Directive` 后缀（后缀可自定义）。

> [no-access-missing-member](http://codelyzer.com/rules/no-access-missing-member/)

不允许在模板中使用组件中不存在的属性和方法。

> [templates-use-public](http://codelyzer.com/rules/templates-use-public/)

模板中只能使用 public 属性和方法。

> [invoke-injectable](http://codelyzer.com/rules/invoke-injectable/)

服务必须使用 `@Injectable()` 装饰。


```