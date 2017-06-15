# 依赖追踪
本节中介绍 Vue 的 Computed 实现机制。示例代码已完成，文本介绍内容 TODO

``` js
/**
 * computed 实现机制
 * 1. computed 初次求值（被 get）时，将全局变量 Dep.target 标记为自身
 * 2. 在该 getter 中使用 computed 函数计算 value
 * 3. computed 函数中包含了对各 reactive 的引用
 * 4. 在该 computed 中对 reactive 求值（触发 getter）时，各 reactive 收集当前 Dep.target 至自身的 deps 数组。computed 求值完成后，所有它所依赖的 reactive 均完成对该 computed 的依赖收集
 * 6. 求值完成后，computed 将 Dep.target 标记为空，返回求值结果作为 computed 的值
 * 7. 下次 reactive 更新时，所有依赖它的 computed 在该 reactive 的 setter 中一并更新
 */

// 标记当前依赖的全局单例
// 由 computed 的 getter 触发时设入该值
// 在 computed 求值期间，computed 依赖的 reactive 通过该值收集该 computed 至其依赖中
// 当 computed 求值完成后再将此值置空
const Dep = {
  target: null
}

// 通过设置 getter 与 setter 定义 computed
// computeFn 为通过 reactive 求值出 computed 的求值函数
// updateCallback 为 computed 更新时触发的回调
function defineComputed (obj, key, computeFn, updateCallback) {
  // 封装供 reactive 收集的更新回调
  // 用于触发 computed 的更新事件
  const onDependencyUpdated = function () {
    // 在此计算出的值没有用于更新 computed
    // 而是用于触发 computed 的更新事件
    // 供后续可能的 watch 等模块使用
    const value = computeFn()
    updateCallback(value)
  }
  Object.defineProperty(obj, key, {
    get () {
      // 标记当前依赖，供 reactive 收集
      Dep.target = onDependencyUpdated
      // 在 computeFn 中对 reactive 求值时
      // 由 reactive 的 setter 收集当前 computed 依赖
      // 故完成对 computed 的初次求值时，即完成了 reactive 的依赖收集
      const value = computeFn()
      // 完成求值后，清空标记
      Dep.target = null
      // 最终返回的 getter 结果
      return value
    },
    // 计算属性无法 set
    set () {}
  })
}

// 通过 getter 与 setter 定义出一个 reactive
function defineReactive (obj, key, val) {
  // 在此标记哪些 computed 依赖了该 reactive
  const deps = []

  Object.defineProperty(obj, key, {
    // 为 reactive 求值时，收集其依赖
    get () {
      if (Dep.target) deps.push(Dep.target)
      // 返回 val 值作为 getter 求值结果
      return val
    },
    // 为 reactive 赋值时，更新所有依赖它的计算属性
    set (newValue) {
      // 在 setter 中更新值
      val = newValue
      // 更新值后触发所有 computed 依赖更新
      deps.forEach((changeFn) => {
        changeFn()
      })
    }
  })
}

const person = {}

defineComputed(
  person,
  'status',
  () => person.age >= 18 ? 'adult' : 'child',
  (newValue) => console.log('Status is', person.status)
)

defineReactive(person, 'age', 22)

person.age = 10
console.log(person.status)
person.age = 30
```
