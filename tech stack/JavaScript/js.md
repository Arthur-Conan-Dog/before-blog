# JavaScript

ref: [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)

## JS as a programming language

多范式语言，既可以 OO 也可以 functional。

1. [Scripting Language vs Programming Language](https://stackoverflow.com/questions/17253545/scripting-language-vs-programming-language)

Scripting languages are often interpreted(解释), rather than compiled(编译).

2. Why JavaScript is a scripting language?

主流观点是如此。依据有如：不像 programming language 一样有静态类型，类型是动态的；或者可以产生可移植的二进制码便于分发或者调用等。

不过 You Don't Know JS Yet 作者认为 JS 是 compiled language。理由是，正因为它有预先编译的过程，我们才能在程序运行前得到一些静态错误反馈，比如 Syntax Error，跟传统的脚本语言不一样。

3. How JS Engine works?

=> JS engine parses the code to an AST.

=> converts AST to a kind-of byte code, a binary intermediate representation (IR), which is then refined/converted even further by the optimizing JIT(Just-In-Time) compiler.

=> JS VM executes the program.

## Scope & Closures

1. what is the scope?

当写程序的时候，JS 如何知道当前有哪些变量可以访问？如果有重名变量应该选用哪一个？这些问题的答案都是通过作用域下的规则来确定的。scope => lexical scope，完全由函数、代码块、变量相对于彼此的放置关系来确定。

Scope 是在编译时被确定的，一般来说（特殊方式有如 eval）不会受到运行时的影响。

对于 compiler 来说，除了声明外，程序中所有出现的变量 / 标识符都属于下面两种角色中的一种：target / source。

*更进一步的解释*：想象 JS Engine，Compiler, (Global) Scope Manager 之间的对话。

compiling: compiler <=> scope manager, asks it if this declaration has already been seen.

execution: engine <=> scope manager, fetch values by declaration.

2. scope chain?

When a function (declaration or expression) is defined, a new scope is created. The positioning of scopes nested inside one another creates a natural scope hierarchy throughout the program, called the scope chain. The scope chain controls variable access, directionally oriented upward and outward.

shadowing: 不同 scope 间同名的变量，会取离当前 scope 最靠近的那个。

arrow functions have the same lexical scope rules as `function` functions do.

3. global scope?

最上层。包含：JS its built-ins; environment hosting the JS engine its own built-ins.

global this(eg. window, global, self) = reference to the global scope object.

4. hoist?

a variable being visible from the beginning of its enclosing scope, even though its declaration may appear further down in the scope.

function hoisting first(only when declared via `function A() {}`), then variable hoisting(only when using `var`).

5. closure?

闭包是一个函数实例，即使该函数被传递到其他范围内调用，它也会记住自己被声明时的外部变量。即使闭包是基于编译时的词法作用域构建，它仍然是函数实例的运行时特征，因为每个不同的函数实例都对应一个不同的闭包。闭包记住的是变量的 ref 而不是快照。

> Closure is observed when a function uses variable(s) from outer scope(s) even while running in a scope where those variable(s) wouldn't be accessible.

概念上来说，每个闭包都对应一个自己的 scope，以及自己的一套外部变量。

被闭包记住的变量不会马上被 GC 回收，只有当闭包函数的实例无人再消费时，才会被回收。

- Pros: 记住之前运行的信息，而不用每次再执行一遍。内部细节被隐藏。

- Cons: 可能造成过多不必要的空间占用。

常见的闭包使用场景：react hooks; currying(redux-middleware).

_currying: closure keeps a function instance alive, or to say, delay its execution. eg. add(1)(2) = 3._

## this & prototype

1. what is ``this``?

> this is neither a reference to the function itself, nor is it a reference to the function's lexical scope.

> this is actually a binding that is made when a function is invoked, and what it references is determined entirely by the call-site where the function is called.

既不是函数自身，也不是函数词法作用域的引用。只是函数被调用时的一个绑定 (runtime binding)，它指向什么完全取决于函数的调用方。

2. how to determine what thing ``this`` is currently binging to?

1) new binding: Called with new? Use the newly constructed object.

2) explicit binding: Called with call or apply (or bind)? Use the specified object.

3) implicit binding: Called with a context object owning the call? Use that context object.

4) default binding: undefined in strict mode, global object otherwise.

5) exception: arrow functions use lexical scoping for ``this`` binding. The lexical binding of an arrow-function cannot be overridden (even with ``new``). eg. equals to declare ``var self = this;`` in advance, and use self in actual function implementations.

3. call, apply, bind?

call & apply: immediately. separate parameters, parameter array.

bind: later.

4. implement your own call, apply & bind?

- [simulate call & apply](https://github.com/mqyqingfeng/Blog/issues/11)

- [simulate bind](https://github.com/mqyqingfeng/Blog/issues/12)

5. [prototype?](https://github.com/mqyqingfeng/Blog/issues/2)

function can be used as a constructor function with ``new``.

fn.prototype: 构造函数所生产的实例的原型对象。

fn.prototype.constructor: 指回构造函数，即 fn.prototype.constructor === fn.

objCreatedByNewFn.\__proto\__: 指向这个对象实例所对应的原型对象，即 objCreatedByNewFn.\__proto\__ === fn.prototype.

fn.prototype.\__proto\__(generally): 原型对象也是一个对象，通过 Object.create / new Object() 创建得到，即 fn.prototype.\__proto\__ === Object.prototype.

Object.prototype.\__proto\__ === null

另外比较特殊的地方是：

fn 本身作为一个函数对象，其原型是什么？fn.\__proto\__ === Function.prototype;

Function 本身作为一个函数对象，其原型是什么？Function.\__proto\__ === Function.prototype;

Object 本身作为一个函数对象，其原型是什么？Object.\__proto\__ === Function.prototype;

6. [what does ``new`` do?](https://github.com/mqyqingfeng/Blog/issues/13)

create a new object instance, bind it as `this` of constructorFn, refer obj.\__proto\__ to constructorFn.prototype.

special returns: if return value is an object, it will overlay the new object; if return value of basic type, this return value will be ignored.

7. [ways of inheritance in JS, pros & cons](https://github.com/mqyqingfeng/Blog/issues/16)

1) 原型链继承。Child.prototype = new Parent(); => 每个实例间共享属性。

2) 构造函数经典继承。Child 构造函数中调用 Parent。=> 每个实例之间的引用属性不共享，但函数方法作为属性也都各自创建了一份造成了重复。

3) 组合继承。=> 引用属性不共享，函数方法共享。但会调用两次父构造函数，实例和 prototype 上的属性有重复。

4) 原型式继承。将传入的对象作为创建的对象的原型。=> 每个实例间共享属性。

> eg. The Object.create() method creates a new object, using an existing object as the prototype of the newly created object.

5) 寄生式继承。

6) 组合寄生式继承。不使用 Child.prototype = new Parent() ，间接让 Child.prototype 能够链到 Parent.prototype. => 只调用了一次 Parent 构造函数，避免了在 Parent.prototype 上创建多余属性，原型链保持不变，能够正常使用 instanceOf & isPrototypeOf。

## JS engine

classic compiler theory: tokenizing/lexing 分词 => parsing 构建 AST => code generation 生成可执行代码

JS engine 的实现要更复杂，还会包括优化代码执行效率的部分，比如回收不必要的变量等。
