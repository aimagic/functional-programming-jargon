# Functional Programming Jargon

Functional programming (FP) provides many advantages, and its popularity has been increasing as a result. However, each programming paradigm comes with its own unique jargon and FP is no exception. By providing a glossary, we hope to make learning FP easier.

Examples are presented in JavaScript (ES2015). [Why JavaScript?](https://github.com/hemanth/functional-programming-jargon/wiki/Why-JavaScript%3F)

*This is a [WIP](https://github.com/hemanth/functional-programming-jargon/issues/20); please feel free to send a PR ;)*

Where applicable, this document uses terms defined in the [Fantasy Land spec](https://github.com/fantasyland/fantasy-land)

__Translations__
* [Portuguese](https://github.com/alexmoreno/jargoes-programacao-funcional)
* [Spanish](https://github.com/idcmardelplata/functional-programming-jargon/tree/master)
* [Chinese](https://github.com/shfshanyue/fp-jargon-zh)
* [Bahasa Indonesia](https://github.com/wisn/jargon-pemrograman-fungsional)
* [Scala World](https://github.com/ikhoon/functional-programming-jargon.scala)
* [Korean](https://github.com/sphilee/functional-programming-jargon)

__Table of Contents__
<!-- RM(noparent,notop) -->

* [Arity](#arity)
* [Higher-Order Functions (HOF)](#higher-order-functions-hof)
* [Closure](#closure)
* [Partial Application](#partial-application)
* [Currying](#currying)
* [Auto Currying](#auto-currying)
* [Function Composition](#function-composition)
* [Continuation](#continuation)
* [Purity](#purity)
* [Side effects](#side-effects)
* [Idempotent](#idempotent)
* [Point-Free Style](#point-free-style)
* [Predicate](#predicate)
* [Contracts](#contracts)
* [Category](#category)
* [Value](#value)
* [Constant](#constant)
* [Functor](#functor)
* [Pointed Functor](#pointed-functor)
* [Lift](#lift)
* [Referential Transparency](#referential-transparency)
* [Equational Reasoning](#equational-reasoning)
* [Lambda](#lambda)
* [Lambda Calculus](#lambda-calculus)
* [Lazy evaluation](#lazy-evaluation)
* [Monoid](#monoid)
* [Monad](#monad)
* [Comonad](#comonad)
* [Applicative Functor](#applicative-functor)
* [Morphism](#morphism)
  * [Endomorphism](#endomorphism)
  * [Isomorphism](#isomorphism)
* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Foldable](#foldable)
* [Lens](#lens)
* [Type Signatures](#type-signatures)
* [Algebraic data type](#algebraic-data-type)
  * [Sum type](#sum-type)
  * [Product type](#product-type)
* [Option](#option)
* [Functional Programming Libraries in JavaScript](#functional-programming-libraries-in-javascript)


<!-- /RM -->

## Arity

[元数](https://en.wikipedia.org/wiki/Arity):
函数接受的参数的个数

The number of arguments a function takes. From words like unary, binary, ternary, etc. This word has the distinction of being composed of two suffixes, "-ary" and "-ity." Addition, for example, takes two arguments, and so it is defined as a binary function or a function with an arity of two. Such a function may sometimes be called "dyadic" by people who prefer Greek roots to Latin. Likewise, a function that takes a variable number of arguments is called "variadic," whereas a binary function must be given two and only two arguments, currying and partial application notwithstanding (see below).

```js
const sum = (a, b) => a + b

const arity = sum.length
console.log(arity) // 2

// The arity of sum is 2
```

## Higher-Order Functions (HOF)

[高阶函数](https://en.wikipedia.org/wiki/Higher-order_function):
接受函数作为参数, 或返回值是函数的函数

A function which takes a function as an argument and/or returns a function.

```js
const filter = (predicate, xs) => xs.filter(predicate)
```

```js
const is = (type) => (x) => Object(x) instanceof type
```

```js
filter(is(Number), [0, '1', 2, null]) // [0, 2]
```

## Closure

[闭包](https://en.wikipedia.org/wiki/Closure_(computer_programming)):
一个作用域, 保留着函数创建时可见的变量

A closure is a scope which retains variables available to a function when it's created. This is important for
[partial application](#partial-application) to work.


```js
const addTo = (x) => {
    return (y) => {
        return x + y
    }
}
```

We can call `addTo` with a number and get back a function with a baked-in `x`.

```js
var addToFive = addTo(5)
```

In this case the `x` is retained in `addToFive`'s closure with the value `5`. We can then call `addToFive` with the `y`
and get back the desired number.

```
addToFive(3) // => 8
```

This works because variables that are in parent scopes are not garbage-collected as long as the function itself is retained.

Closures are commonly used in event handlers so that they still have access to variables defined in their parents when they
are eventually called.

__Further reading__
* [Lambda Vs Closure](http://stackoverflow.com/questions/220658/what-is-the-difference-between-a-closure-and-a-lambda)
* [How do JavaScript Closures Work?](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)


## Partial Application

[部分应用](https://en.wikipedia.org/wiki/Partial_application):
为原函数创建一个新函数, 指定部分参数

Partially applying a function means creating a new function by pre-filling some of the arguments to the original function.


```js
// Helper to create partially applied functions
// Takes a function and some arguments
const partial = (f, ...args) =>
  // returns a function that takes the rest of the arguments
  (...moreArgs) =>
    // and calls the original function with all of them
    f(...args, ...moreArgs)

// Something to apply
const add3 = (a, b, c) => a + b + c

// Partially applying `2` and `3` to `add3` gives you a one-argument function
const fivePlus = partial(add3, 2, 3) // (c) => 2 + 3 + c

fivePlus(4) // 9
```

You can also use `Function.prototype.bind` to partially apply a function in JS:

```js
const add1More = add3.bind(null, 2, 3) // (c) => 2 + 3 + c
```

Partial application helps create simpler functions from more complex ones by baking in data when you have it. [Curried](#currying) functions are automatically partially applied.


## Currying

[柯里化](https://en.wikipedia.org/wiki/Currying):
将接受多个参数的函数, 转化为接受单个参数的函数

The process of converting a function that takes multiple arguments into a function that takes them one at a time.

Each time the function is called it only accepts one argument and returns a function that takes one argument until all arguments are passed.

```js
const sum = (a, b) => a + b

const curriedSum = (a) => (b) => a + b

curriedSum(40)(2) // 42.

const add2 = curriedSum(2) // (b) => 2 + b

add2(10) // 12

```

## Auto Currying

[自动柯里化]():
将接受多个参数的函数, 转化为接受单个参数的函数, 若给的参数个数不够, 则转化为接受剩余参数的函数

Transforming a function that takes multiple arguments into one that if given less than its correct number of arguments returns a function that takes the rest. When the function gets the correct number of arguments it is then evaluated.

lodash & Ramda have a `curry` function that works this way.

```js
const add = (x, y) => x + y

const curriedAdd = _.curry(add)
curriedAdd(1, 2) // 3
curriedAdd(1) // (y) => 1 + y
curriedAdd(1)(2) // 3
```

__Further reading__
* [Favoring Curry](http://fr.umio.us/favoring-curry/)
* [Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)

## Function Composition

[函数组合](https://en.wikipedia.org/wiki/Function_composition_(computer_science)):
将两个函数, 其中一个的输出作为另一个的输入, 组合为一个新函数

The act of putting two functions together to form a third function where the output of one function is the input of the other.

```js
const compose = (f, g) => (a) => f(g(a)) // Definition
const floorAndToString = compose((val) => val.toString(), Math.floor) // Usage
floorAndToString(121.212121) // '121'
```

## Continuation

[Continuation](https://en.wikipedia.org/wiki/Continuation):
某时刻, 有待执行的一部分代码

At any given point in a program, the part of the code that's yet to be executed is known as a continuation.

```js
const printAsString = (num) => console.log(`Given ${num}`)

const addOneAndContinue = (num, cc) => {
  const result = num + 1
  cc(result)
}

addOneAndContinue(2, printAsString) // 'Given 3'
```

Continuations are often seen in asynchronous programming when the program needs to wait to receive data before it can continue. The response is often passed off to the rest of the program, which is the continuation, once it's been received.

```js
const continueProgramWith = (data) => {
  // Continues program with data
}

readFileAsync('path/to/file', (err, response) => {
  if (err) {
    // handle error
    return
  }
  continueProgramWith(response)
})
```

## Purity

[纯度](https://en.wikipedia.org/wiki/Pure_function):
若函数的返回值仅取决于输入值, 不产生副作用, 则称为纯函数

A function is pure if the return value is only determined by its
input values, and does not produce side effects.

```js
const greet = (name) => `Hi, ${name}`

greet('Brianne') // 'Hi, Brianne'
```

As opposed to each of the following:

```js
window.name = 'Brianne'

const greet = () => `Hi, ${window.name}`

greet() // "Hi, Brianne"
```

The above example's output is based on data stored outside of the function...

```js
let greeting

const greet = (name) => {
  greeting = `Hi, ${name}`
}

greet('Brianne')
greeting // "Hi, Brianne"
```

... and this one modifies state outside of the function.

## Side effects

[副作用](https://en.wikipedia.org/wiki/Side_effect_(computer_science)):
一个函数或表达式, 除返回值之外, 与外部可变状态产生了交互, 则称其有副作用

A function or expression is said to have a side effect if apart from returning a value, it interacts with (reads from or writes to) external mutable state.

```js
const differentEveryTime = new Date()
```

```js
console.log('IO is a side effect!')
```

## Idempotent

[幂等](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning):
一个函数, 若将其结果作为输入, 结果不变, 则称之为幂等的

A function is idempotent if reapplying it to its result does not produce a different result.

```
f(f(x)) ≍ f(x)
```

```js
Math.abs(Math.abs(10))
```

```js
sort(sort(sort([2, 1])))
```

## Point-Free Style

[Point-Free Style](https://en.wikipedia.org/wiki/Tacit_programming):
不显式指定函数参数的风格

Writing functions where the definition does not explicitly identify the arguments used. This style usually requires [currying](#currying) or other [Higher-Order functions](#higher-order-functions-hof). A.K.A Tacit programming.

```js
// Given
const map = (fn) => (list) => list.map(fn)
const add = (a) => (b) => a + b

// Then

// Not points-free - `numbers` is an explicit argument
const incrementAll = (numbers) => map(add(1))(numbers)

// Points-free - The list is an implicit argument
const incrementAll2 = map(add(1))
```

`incrementAll` identifies and uses the parameter `numbers`, so it is not points-free.  `incrementAll2` is written just by combining functions and values, making no mention of its arguments.  It __is__ points-free.

Points-free function definitions look just like normal assignments without `function` or `=>`.

## Predicate

[谓词](https://en.wikipedia.org/wiki/Predicate_(mathematical_logic))
一个函数, 对于给定的参数, 返回相应的 true 或 false

A predicate is a function that returns true or false for a given value. A common use of a predicate is as the callback for array filter.

```js
const predicate = (a) => a > 2

;[1, 2, 3, 4].filter(predicate) // [3, 4]
```

## Contracts

[契约]():
约定某种表现

A contract specifies the obligations and guarantees of the behavior from a function or expression at runtime. This acts as a set of rules that are expected from the input and output of a function or expression, and errors are generally reported whenever a contract is violated.

```js
// Define our contract : int -> int
const contract = (input) => {
  if (typeof input === 'number') return true
  throw new Error('Contract violated: expected int -> int')
}

const addOne = (num) => contract(num) && num + 1

addOne(2) // 3
addOne('some string') // Contract violated: expected int -> int
```

## Category

[范畴](https://en.wikipedia.org/wiki/Category_(mathematics)):
(范畴论)对象和其态射的集合

A category in category theory is a collection of objects and morphisms between them. In programming, typically types
act as the objects and functions as morphisms.

To be a valid category 3 rules must be met:

1. There must be an identity morphism that maps an object to itself.
    Where `a` is an object in some category,
    there must be a function from `a -> a`.
2. Morphisms must compose.
    Where `a`, `b`, and `c` are objects in some category,
    and `f` is a morphism from `a -> b`, and `g` is a morphism from `b -> c`;
    `g(f(x))` must be equivalent to `(g • f)(x)`.
3. Composition must be associative
    `f • (g • h)` is the same as `(f • g) • h`

Since these rules govern composition at very abstract level, category theory is great at uncovering new ways of composing things.

__Further reading__

* [Category Theory for Programmers](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)

## Value

[值](https://en.wikipedia.org/wiki/Value_(computer_science)):
任何可以赋给变量的东西

Anything that can be assigned to a variable.

```js
5
Object.freeze({name: 'John', age: 30}) // The `freeze` function enforces immutability.
;(a) => a
;[1]
undefined
```

## Constant

[常量](https://en.wikipedia.org/wiki/Constant_(computer_programming)):
不能再赋值的变量

A variable that cannot be reassigned once defined.

```js
const five = 5
const john = Object.freeze({name: 'John', age: 30})
```

Constants are [referentially transparent](#referential-transparency). That is, they can be replaced with the values that they represent without affecting the result.

With the above two constants the following expression will always return `true`.

```js
john.age + five === ({name: 'John', age: 30}).age + (5)
```

## Functor

[函子]():
一个对象, 实现了 `map` 方法

An object that implements a `map` function which, while running over each value in the object to produce a new object, adheres to two rules:

__Preserves identity__
```
object.map(x => x) ≍ object
```

__Composable__

```
object.map(compose(f, g)) ≍ object.map(g).map(f)
```

(`f`, `g` are arbitrary functions)

A common functor in JavaScript is `Array` since it abides to the two functor rules:

```js
;[1, 2, 3].map(x => x) // = [1, 2, 3]
```

and

```js
const f = x => x + 1
const g = x => x * 2

;[1, 2, 3].map(x => f(g(x))) // = [3, 5, 7]
;[1, 2, 3].map(g).map(f)     // = [3, 5, 7]
```

## Pointed Functor

[Pointed Functor]():
一个函子, 实现了 `of` 方法

An object with an `of` function that puts _any_ single value into it.

ES2015 adds `Array.of` making arrays a pointed functor.

```js
Array.of(1) // [1]
```

## Lift

[提升](https://en.wikipedia.org/wiki/Lift_(mathematics)):
将一个值置入一个函子这类对象

Lifting is when you take a value and put it into an object like a [functor](#pointed-functor). If you lift a function into an [Applicative Functor](#applicative-functor) then you can make it work on values that are also in that functor.

Some implementations have a function called `lift`, or `liftA2` to make it easier to run functions on functors.

```js
const liftA2 = (f) => (a, b) => a.map(f).ap(b) // note it's `ap` and not `map`.

const mult = a => b => a * b

const liftedMult = liftA2(mult) // this function now works on functors like array

liftedMult([1, 2], [3]) // [3, 6]
liftA2(a => b => a + b)([1, 2], [3, 4]) // [4, 5, 5, 6]
```

Lifting a one-argument function and applying it does the same thing as `map`.

```js
const increment = (x) => x + 1

lift(increment)([2]) // [3]
;[2].map(increment) // [3]
```


## Referential Transparency

[引用透明性](https://en.wikipedia.org/wiki/Referential_transparency):
一个表达式, 可以替换为其值, 而不用更改程序的行为, 称为引用透明的

An expression that can be replaced with its value without changing the
behavior of the program is said to be referentially transparent.

Say we have function greet:

```js
const greet = () => 'Hello World!'
```

Any invocation of `greet()` can be replaced with `Hello World!` hence greet is
referentially transparent.

##  Equational Reasoning

[等值推理](https://en.wikipedia.org/wiki/Universal_algebra):
若一个程序是表达式的组合, 且没有副作用, 则系统的真假可由其一部分得出

When an application is composed of expressions and devoid of side effects, truths about the system can be derived from the parts.

## Lambda

[拉姆达](https://en.wikipedia.org/wiki/Anonymous_function):
匿名函数, 可以当作值

An anonymous function that can be treated like a value.

```js
;(function (a) {
  return a + 1
})

;(a) => a + 1
```
Lambdas are often passed as arguments to Higher-Order functions.

```js
;[1, 2].map((a) => a + 1) // [2, 3]
```

You can assign a lambda to a variable.

```js
const add1 = (a) => a + 1
```

## Lambda Calculus

[拉姆达演算](https://en.wikipedia.org/wiki/Lambda_calculus):
数理逻辑中的一个形式系统, 基于函数抽象和使用变量绑定和替换的应用

A branch of mathematics that uses functions to create a [universal model of computation](https://en.wikipedia.org/wiki/Lambda_calculus).

## Lazy evaluation

[惰性求值](https://en.wikipedia.org/wiki/Lazy_evaluation):
一种按需调用求值机制, 将表达式求值延迟到需要其值时

Lazy evaluation is a call-by-need evaluation mechanism that delays the evaluation of an expression until its value is needed. In functional languages, this allows for structures like infinite lists, which would not normally be available in an imperative language where the sequencing of commands is significant.

```js
const rand = function*() {
  while (1 < 2) {
    yield Math.random()
  }
}
```

```js
const randIter = rand()
randIter.next() // Each execution gives a random value, expression is evaluated on need.
```

## Monoid

[幺半群](https://en.wikipedia.org/wiki/Monoid):
一个对象, 带一个函数, 该函数可以"结合"该对象与另一个同类对象

An object with a function that "combines" that object with another of the same type.

一个简单的幺半群, 数的相加:

One simple monoid is the addition of numbers:

```js
1 + 1 // 2
```

此例中, 数是对象, `+` 是函数

In this case number is the object and `+` is the function.

( 幺元, 单位元 Identity )必须存在一个 "幺元" 值, 当结合其它值时, 不会改变那些值

An "identity" value must also exist that when combined with a value doesn't change it.

对于相加来说, 幺元值为 `0`

The identity value for addition is `0`.
```js
1 + 0 // 1
```

( 结合律 Associativity ) 操作分组必须不影响结果:

It's also required that the grouping of operations will not affect the result (associativity):

```js
1 + (2 + 3) === (1 + 2) + 3 // true
```

数组连接也形成一个幺半群:

Array concatenation also forms a monoid:

```js
;[1, 2].concat([3, 4]) // [1, 2, 3, 4]
```

其幺元值为空数组 `[]`

The identity value is empty array `[]`

```js
;[1, 2].concat([]) // [1, 2]
```

若提供幺元和组合函数, 则函数自身形成一个幺半群:

If identity and compose functions are provided, functions themselves form a monoid:

```js
const identity = (a) => a
const compose = (f, g) => (x) => f(g(x))
```

`foo` 是接受一个参数的任意函数

`foo` is any function that takes one argument.

```
compose(foo, identity) ≍ compose(identity, foo) ≍ foo
```

## Monad

~~Monad 说白了不过就是自函子范畴上的一个幺半群~~

[单子](https://en.wikipedia.org/wiki/Monad_(functional_programming)):
一个对象, 带 [`of`](#pointed-functor) 和 `chain` 函数. 
`chain` 类似 [`map`](#functor), 除了它会为嵌套对象结果解除嵌套

A monad is an object with [`of`](#pointed-functor) and `chain` functions. `chain` is like [`map`](#functor) except it un-nests the resulting nested object.

```js
// Implementation
Array.prototype.chain = function (f) {
  return this.reduce((acc, it) => acc.concat(f(it)), [])
}

// Usage
Array.of('cat,dog', 'fish,bird').chain((a) => a.split(',')) // ['cat', 'dog', 'fish', 'bird']

// Contrast to map
Array.of('cat,dog', 'fish,bird').map((a) => a.split(',')) // [['cat', 'dog'], ['fish', 'bird']]
```

在其它函数式语言中, `of` 也称为 `return`.
在其它语言中, `chain` 也称为 `flatmap` 和 `bind`

`of` is also known as `return` in other functional languages.
`chain` is also known as `flatmap` and `bind` in other languages.

## Comonad

[余单子](https://en.wikipedia.org/wiki/Monad_(functional_programming)#Comonads):
一个对象, 有 `extract` 和 `extend` 函数

An object that has `extract` and `extend` functions.

```js
const CoIdentity = (v) => ({
  val: v,
  extract () {
    return this.val
  },
  extend (f) {
    return CoIdentity(f(this))
  }
})
```

`extract` 从函子中提取一个值

Extract takes a value out of a functor.

```js
CoIdentity(1).extract() // 1
```

`extend` 在余单子上跑一个函数. 该函数返回类型应与余单子相同

Extend runs a function on the comonad. The function should return the same type as the comonad.

```js
CoIdentity(1).extend((co) => co.extract() + 1) // CoIdentity(2)
```

## Applicative Functor

[可应用函子](https://en.wikipedia.org/wiki/Applicative_functor):
一个对象, 带 `ap` 函数. `ap` 把一个对象中的函数应用到另一同类对象中的值

An applicative functor is an object with an `ap` function. `ap` applies a function in the object to a value in another object of the same type.

```js
// 实现
Array.prototype.ap = function (xs) {
  return this.reduce((acc, f) => acc.concat(xs.map(f)), [])
}

// 用法示例
;[(a) => a + 1].ap([1]) // [2]
```

其作用体现在, 若有两个对象, 想要应用一个二元函数到它们的内容

This is useful if you have two objects and you want to apply a binary function to their contents.

```js
// 想要组合的数组
const arg1 = [1, 3]
const arg2 = [4, 5]

// 组合函数, 这里必须柯里化才能用
const add = (x) => (y) => x + y

const partiallyAppliedAdds = [add].ap(arg1) // [(y) => 1 + y, (y) => 3 + y]
```

产生了一个函数数组, 可以在之上调用 `ap` 得到结果:

This gives you an array of functions that you can call `ap` on to get the result:

```js
partiallyAppliedAdds.ap(arg2) // [5, 6, 7, 8]
```

## Morphism

[态射](https://en.wikipedia.org/wiki/Morphism):
一个转化函数

A transformation function.

### Endomorphism

[自同态](https://en.wikipedia.org/wiki/Endomorphism):
一个函数, 其输入类型与输出类型相同

A function where the input type is the same as the output.

```js
// uppercase :: String -> String
const uppercase = (str) => str.toUpperCase()

// decrement :: Number -> Number
const decrement = (x) => x - 1
```

### Isomorphism

[同构](https://en.wikipedia.org/wiki/Isomorphism):
两类对象之间的一对转化, 结构不变, 数据无损

A pair of transformations between 2 types of objects that is structural in nature and no data is lost.

例如, 2D 坐标可存储为数组 `[2,3]` 或对象 `{x: 2, y: 3}`

For example, 2D coordinates could be stored as an array `[2,3]` or object `{x: 2, y: 3}`.

```js
// 提供双向转换函数使它们成为同构的 (isomorphic)
const pairToCoords = (pair) => ({x: pair[0], y: pair[1]})
const coordsToPair = (coords) => [coords.x, coords.y]

coordsToPair(pairToCoords([1, 2])) // [1, 2]
pairToCoords(coordsToPair({x: 1, y: 2})) // {x: 1, y: 2}
```

## Setoid

[Setoid](https://en.wikipedia.org/wiki/Setoid):
一个带 `equals` 函数的对象, `equals` 用于和其它同类对象比较

An object that has an `equals` function which can be used to compare other objects of the same type.

让数组成为 setoid:

Make array a setoid:

```js
Array.prototype.equals = function (arr) {
  const len = this.length
  if (len !== arr.length) {
    return false
  }
  for (let i = 0; i < len; i++) {
    if (this[i] !== arr[i]) {
      return false
    }
  }
  return true
}

;[1, 2].equals([1, 2]) // true
;[1, 2].equals([0]) // false
```

## Semigroup

[半群](https://en.wikipedia.org/wiki/Semigroup):
一个带 `concat` 函数的对象, `concat` 可将该对象与另一同类型对象组合

An object that has a `concat` function that combines it with another object of the same type.

```js
;[1].concat([2]) // [1, 2]
```

## Foldable

[可折叠](https://en.wikipedia.org/wiki/Fold_(higher-order_function)):
一个带 `reduce` 函数的对象, `reduce` 可将该对象转化为其它类型

An object that has a `reduce` function that can transform that object into some other type.

```js
const sum = (list) => list.reduce((acc, val) => acc + val, 0)
sum([1, 2, 3]) // 6
```

## Lens ##

[透镜]():
一个结构 (通常是一个对象或函数), 有一对 getter 和不可变 setter 用于其它数据结构

A lens is a structure (often an object or function) that pairs a getter and a non-mutating setter for some other data structure.

```js
// 用到 [Ramda 的透镜](http://ramdajs.com/docs/#lens)
const nameLens = R.lens(
  // getter for name property on an object
  (obj) => obj.name,
  // setter for name property
  (val, obj) => Object.assign({}, obj, {name: val})
)
```

一个数据结构有了这一对 get 和 set, 就有了一些关键特性
Having the pair of get and set for a given data structure enables a few key features.

```js
const person = {name: 'Gertrude Blanch'}

// 调用 getter
R.view(nameLens, person) // 'Gertrude Blanch'

// 调用 setter
R.set(nameLens, 'Shafi Goldwasser', person) // {name: 'Shafi Goldwasser'}

// 在结构的值上跑一个函数
R.over(nameLens, uppercase, person) // {name: 'GERTRUDE BLANCH'}
```

透镜也是可组合的. 这简化了嵌套数据的深度不可变更新

Lenses are also composable. This allows easy immutable updates to deeply nested data.

```js
// This lens focuses on the first item in a non-empty array
const firstLens = R.lens(
  // get first item in array
  xs => xs[0],
  // non-mutating setter for first item in array
  (val, [__, ...xs]) => [val, ...xs]
)

const people = [{name: 'Gertrude Blanch'}, {name: 'Shafi Goldwasser'}]

// Despite what you may assume, lenses compose left-to-right.
R.over(compose(firstLens, nameLens), uppercase, people) // [{'name': 'GERTRUDE BLANCH'}, {'name': 'Shafi Goldwasser'}]
```

其它实现:

* [partial.lenses](https://github.com/calmm-js/partial.lenses) - Tasty syntax sugar and a lot of powerful features
* [nanoscope](http://www.kovach.me/nanoscope/) - Fluent-interface

## Type Signatures

[类型签名](https://en.wikipedia.org/wiki/Type_signature):
通常 JavaScript 中的函数有注释说明其参数和返回值的类型.

Often functions in JavaScript will include comments that indicate the types of their arguments and return values.

社区虽不一致, 但常用如下模式:

There's quite a bit of variance across the community but they often follow the following patterns:

```js
// functionName :: firstArgType -> secondArgType -> returnType

// add :: Number -> Number -> Number
const add = (x) => (y) => x + y

// increment :: Number -> Number
const increment = (x) => x + 1
```

若函数接受另一函数作为参数, 则另一函数放在括号里

If a function accepts another function as an argument it is wrapped in parentheses.

```js
// call :: (a -> b) -> a -> b
const call = (f) => (x) => f(x)
```

字母 `a`, `b`, `c`, `d` 用于标示任意类型的参数.
下例中的 `map` 接受:
1. 一个把某类型 `a` 的值转化为另一类型 `b` 的函数
2. 一个 `a` 类型值的数组
并返回一个 `b` 类型值的数组

The letters `a`, `b`, `c`, `d` are used to signify that the argument can be of any type. The following version of `map` takes a function that transforms a value of some type `a` into another type `b`, an array of values of type `a`, and returns an array of values of type `b`.

```js
// map :: (a -> b) -> [a] -> [b]
const map = (f) => (list) => list.map(f)
```

__延伸阅读__
* [Ramda 的类型签名](https://github.com/ramda/ramda/wiki/Type-Signatures)
* [Mostly Adequate Guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html#whats-your-type)
* Stack Overflow 上的 [What is Hindley-Milner?](http://stackoverflow.com/a/399392/22425)

## Algebraic data type

[代数数据类型](https://en.wikipedia.org/wiki/Algebraic_data_type):
一个组合类型, 由其它类型组合而成.
两个常见代数类型有 [和](#sum-type) 与 [积](#product-type)

### Sum type

[和类型](https://en.wikipedia.org/wiki/Tagged_union):
两个类型的合二为一.
因结果类型所有可能的值是输入类型所有可能的值之和, 故而称为和类型

JavaScript 没有这种类型, 但可以用 `Set` 模拟:

```js
// 设想忽略 Set, 这里有某些只能有这些值的类型
const bools = new Set([true, false])
const halfTrue = new Set(['half-true'])

// 该 weakLogic 类型包含 bools 和 halfTrue 的值之和
const weakLogicValues = new Set([...bools, ...halfTrue])
```

Sum type 也称 tagged union, variant, variant record, choice type, discriminated union, disjoint union

有 [几个](https://github.com/paldepind/union-type) JS [库](https://github.com/puffnfresh/daggy) 可以帮助定义和使用 union type

_Flow_ 有 [union types](https://flow.org/en/docs/types/unions/), _TypeScript_ 有 [Enums](https://www.typescriptlang.org/docs/handbook/enums.html) 来干这个

### Product type

[积类型](https://en.wikipedia.org/wiki/Product_type):
将类型以如下形式组合, 我们都熟悉:

A **product** type combines types together in a way you're probably more familiar with:

```js
// point :: (Number, Number) -> {x: Number, y: Number}
const point = (x, y) => ({ x, y })
```

因结果类型所有可能的值是输入类型所有可能的值之积, 故而称为积类型
很多语言的 tuple 类型即是一种简单的积类型

参见 [Set theory](https://en.wikipedia.org/wiki/Set_theory).

## Option

[Option](https://en.wikipedia.org/wiki/Option_type):
一个带 `Some` 和 `None` 的 [和类型](#sum-type)

Option 可用于组合无返回值的函数

```js
// Naive definition

const Some = (v) => ({
  val: v,
  map (f) {
    return Some(f(this.val))
  },
  chain (f) {
    return f(this.val)
  }
})

const None = () => ({
  map (f) {
    return this
  },
  chain (f) {
    return this
  }
})

// maybeProp :: (String, {a}) -> Option a
const maybeProp = (key, obj) => typeof obj[key] === 'undefined' ? None() : Some(obj[key])
```
Use `chain` to sequence functions that return `Option`s
```js

// getItem :: Cart -> Option CartItem
const getItem = (cart) => maybeProp('item', cart)

// getPrice :: Item -> Option Number
const getPrice = (item) => maybeProp('price', item)

// getNestedPrice :: cart -> Option a
const getNestedPrice = (cart) => getItem(cart).chain(getPrice)

getNestedPrice({}) // None()
getNestedPrice({item: {foo: 1}}) // None()
getNestedPrice({item: {price: 9.99}}) // Some(9.99)
```

`Option` is also known as `Maybe`. `Some` is sometimes called `Just`. `None` is sometimes called `Nothing`.

## Functional Programming Libraries in JavaScript

* [mori](https://github.com/swannodette/mori)
* [Immutable](https://github.com/facebook/immutable-js/)
* [Ramda](https://github.com/ramda/ramda)
* [ramda-adjunct](https://github.com/char0n/ramda-adjunct)
* [Folktale](http://folktale.origamitower.com/)
* [monet.js](https://cwmyers.github.io/monet.js/)
* [lodash](https://github.com/lodash/lodash)
* [Underscore.js](https://github.com/jashkenas/underscore)
* [Lazy.js](https://github.com/dtao/lazy.js)
* [maryamyriameliamurphies.js](https://github.com/sjsyrek/maryamyriameliamurphies.js)
* [Haskell in ES6](https://github.com/casualjavascript/haskell-in-es6)
* [Sanctuary](https://github.com/sanctuary-js/sanctuary)
* [Crocks](https://github.com/evilsoft/crocks)

---

__P.S:__ This repo is successful due to the wonderful [contributions](https://github.com/hemanth/functional-programming-jargon/graphs/contributors)!
