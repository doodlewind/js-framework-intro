# MVC 模式

MVC 是 UI 编程领域中非常经典的设计模式，它通过将应用分离出 Model / View / Controller 三个模块，使得开发者能够借助该模式，构建出更易于扩展和维护的应用程序。

在本节中，我们将基于经典的 MVC 理念，实现一个 50 行内的极简 MVC 框架，用于支持一个 Todo App 示例应用的开发。在这个实现过程中，可以理解该模式在提升可维护性和可扩展性上所做出的努力。

## 简介与动机
MVC 模式在概念上，强调 Model / View / Controller 三个模块的分离：

* **Model**: 专注数据的存取，对基础数据对象的封装。
* **Controller**: 处理形如【用户登录】、【验证表单】等具体操作时的业务逻辑，从 Model 中存取数据，并渲染到 View 中。
* **View**: 负责界面视图，可理解为【输入数据，输出界面】的模块，在其中通常不涉及的业务逻辑。

需要注意的是，MVC 仅是一种模式理念，而非具体的规范。因此，根据 MVC 的理念所设计出的框架，在实现和使用上可能存在着较大的区别。下文中介绍的框架，仅是作者作为示例给出的一种实现，供读者参考。

那么，为什么需要 MVC 这一框架级的设计模式呢？相信许多前端开发者都经历过纯粹基于 jQuery / Zepto 等基础类库开发前端项目的阶段。在这种情境下，完全不需要考虑 MVC 的概念，对业务逻辑的实现是非常简单而直接的。以实现【点击某按钮时，从后台接口获取数据，然后渲染到页面】这样非常常见的需求为例，典型的业务代码实现形如：

``` js
// 点击某按钮时
$('.xx-btn').click(() => {
  // 从后台接口获取数据
  $.get('/xx-api', (data) => {
    // 拼接模板
    const template = `<div>${data}</div>`
    // 渲染到页面上
    $('#xx-table').html(template)
  })
})
```

基于这些便捷的 API，确实能在寥寥数行之内实现业务需求。这时，存在的问题包括：

* 处理【用户输入】与【数据渲染】的逻辑，以硬编码的形式结合在了一起，一般项目中数据渲染的代码多直接书写在匿名回调函数内，难以抽离复用（尤其是在回调函数内使用了 `this` 时，更难复用相应的渲染代码）。
* 在没有 ES6 的时代，字符串拼接的代码十分难以维护，各种形如 `'<div class="' + data.class + '">'` 这样引号交错的代码虽然容易符合初学者的直觉写出，但显然难以阅读和维护。
* 将数据直接映射到页面 DOM 上的操作同样很符合初学者的直觉，但也造成应用数据模型的缺乏。在页面【存在多种可能的输入，而每种输入所对应的输出均可直接修改 DOM 结构】时，会带来陡增的状态复杂性。

虽然 MVC 不是银弹，但其出现确实提供了一种解决上述问题的思路。下文中将从一个极简的 MVC 框架所需要具备的特性出发，一步步介绍我们解决上述问题的思路。

## 框架特性
与许多设计模式一样，MVC 在概念定义上较为抽象，通过代码实现掌握它，显然比模糊的文字描述要更为有效。下面作为示例，我们将一步步实现这个名为 NanoMVC 的框架。

在开始编写具体代码之前，需要在设计阶段明确的是，这个框架将提供何种功能，使用者该如何使用。基于使用情景的需求，给出一些简单的示例代码，最后再从满足这些示例的目标出发，进行实际的编码实现。在【先明确需求，再进行编码实现】的这一点上，框架开发与业务逻辑开发是很接近的，这样目标驱动的开发模式也有利于保持开发流程的可控，而非像实现简单业务需求那样随心所欲地【想到什么写什么】，以至于最后产出缺乏架构支持而不易维护的代码。

那么，NanoMVC 框架的使用情景是什么呢？常见的后端框架所封装的功能，不外乎对数据的增查改删与渲染。在前端，我们以一个非常简单的 Todo App 作为框架所需要支持的业务开发场景，这时基于框架的业务代码，所需要实现的功能包括：

* **TodoModel** 模块实现 Todo 这一数据模型的存取。
* **TodoView** 模块实现将 Todo 数据模型渲染到页面。
* **TodoController** 模块实现对 Todo 数据的新增、编辑、删除等操作。

在用于实现 Todo 业务逻辑的 Model 和 Controller 模块，除了具备框架提供的若干功能外，还需要在其中编写特定的业务代码。这时，面向对象的编程范式就能够发挥作用了：通过形如 TodoController 的业务代码【继承】框架所提供的 Controller 基类，在其中即可调用框架所封装的功能。这也就意味着，该框架的功能是需要支持通过继承来使用的。

## 使用示例
在【支持 MVC 三个模块】与【支持通过继承使用】的两个目标确定后，即可以框架使用者的身份，给出调用框架的基础示例代码：

``` js
// Todo 项目的入口 index.js 文件

// 依次导入 MVC 的三个模块
import { TodoModel } from './model'
import { TodoController } from './controller'
import { TodoView as view } from './view'

// 实例化各模块
// 此处 View 的实现由于过于简单而不需实例化
const model = new TodoModel()
const controller = new TodoController(model, view)

// 通过手动调用 render 方法，初始化页面
// 在 controller 中 render 值得商榷
// 但其不失为一种很方便的实现
controller.render()
```

在这个应用的入口模块中，仅仅明确了 MVC 模块拆分的基本结构，而并没有给出各业务模块的基本实现示例。下面给出以框架使用者的角度，在利用框架实现 TodoModel 模块时的基本示例代码：

``` js
// model.js
// 该示例为实际业务代码，并非框架代码

// 从框架中导入 Model 基类
import { Model } from 'framework'

// 通过继承实现自己的业务 Model
export class TodoModel extends Model {
  // 在构造器中定义数据模型的基本结构
  // 在此即为 Todo 列表
  constructor () {
    super({ todos: [] })
  }
  // 利用 ES6 class 语法定义模型实例的 getter
  // 从而在调用 model.todos 时返回正确的 Todo 数据
  get todos () {
    return this.data.todos
  }
  // 利用 ES6 class 语法定义模型实例的 setter
  // 从而在执行形如 modle.todos = newTodos 的赋值时
  // 能够通知订阅了 Model 的模块进行相应更新
  set todos (todos) {
    this.data.todos = todos
    this.publish(todos)
  }
}
```

虽然 Model 模块的有效代码仅有十余行，但其实现中已体现了使用框架的基本方式：

1. 导入框架提供的基类。
2. 继承基类，根据实际业务需求定制出相应的模块实例。

与之类似地，实现 TodoController 模块的业务代码，也按照类似的思路进行组织：

``` js
// controller.js
// 该示例为实际业务代码，并非框架代码

import { Controller } from 'framework'

// 同样基于继承实现业务特定的 Controller
export class TodoController extends Controller {
  constructor (model, view) {
    // 在此提供实例化 Controller 时所需的参数
    // 包括其对应的 Model 与 View，及相应的业务逻辑功能
    super({
      model,
      view,
      el: '#app',
      // 提供点击页面特定按钮时的处理逻辑
      onClick: {
        // 点击【新增】按钮时，新增 Todo
        '.btn-add' () {
          // this.model.todos = ...
        },
        // 点击【删除】按钮时，删除相应的 Todo
        '.btn-delete' (e) {
          // this.model.todos = ...
        },
        // 点击【更新】按钮时，更新相应的 Todo
        '.btn-update' (e) {
          // 根据 id 更新元素
          // this.model.todos = ...
        }
      }
    })
    // 订阅 Model 更新事件
    // 在 Model 更新时执行 controller 的 render 方法
    this.model.subscribers.push(this.render)
  }
}
```

在实现了这个 Controller 后，基本的业务逻辑流程已经初见端倪了：

1. Controller 根据 onClick 参数中提供的字段，**声明式地**定义形如【当 class 名称包含 `btn-add` 的按钮点击时】所触发的业务逻辑，从而避免 `$(...)` 绑定点击事件的底层操作。
2. 在点击事件触发时，**不去直接修改 DOM**，而是更新 `this.model` 中的数据。这样，就实现了数据模型与页面视图的分离解耦。
3. 还记得 `model.js` 中实现的 `this.publish(todos)` 吗？在上一步中数据更新触发数据模型的 setter 时，Model 将通知【数据变更】的事件至模型的所有订阅者。
4. 在 Controller 的最后一行有效代码中，它订阅了数据模型的更新事件。故而在数据更新时，将触发 render 方法以重绘 DOM。

在这个流程中，不仅包含了对业务逻辑相关事件的配置式 / 声明式定义，还实现了发布 - 订阅这一设计模式，该模式也广泛应用在 MVC 框架设计中。而在 Controller 中业务逻辑成功更新后，最后需要的就是更新 View 了。相应的 TodoView 模块示例如下：

``` js
// view.js
// 该模块是一个【输入数据，输出 HTML 模板】的纯函数
// 故而在业务代码中即可实现其全部功能，不需要继承

export function TodoView ({ todos }) {
  // 根据 Todo 数组数据
  // 通过 map 得到带【编辑】与【删除】按钮的单条 Todo 页面模板
  const todosList = todos.map(todo => `
    <div>
      <span>${todo.text}</span>
      <button data-id="${todo.id}" class="btn-delete">
        Delete
      </button>

      <span>
        <input data-id="${todo.id}"/>
        <button data-id="${todo.id}" class="btn-update">
          Update
        </button>
      </span>
    </div>
  `).join('')
  
  // 通过 ES6 模板字符串实现类似 JSX 格式的 View 输出
  return (`
    <main>
      <input class="input-add"/>
      <button class="btn-add">Add</button>
      <div>${todosList}</div>
    </main>
  `)
}
```

到此为止，我们已经勾画出业务代码的全貌了。接下来需要的就是实现 NanoMVC 这一框架，从而让这些代码能够在框架的支持下正常运行。
