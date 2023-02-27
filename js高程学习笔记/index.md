# js高程学习笔记


# 红宝书（学习笔记）

## 三、语言基础

### null 和 undefined（3.4）

- null: 表示一个空对象指针。这也是 typeof null 会返回‘object’的原因。

- undefined： undefined 值由 null 派生而来，所以表面相等（null == undefined // true）

  当声明了变量但未赋值的时候，就相当于给变量赋值了 undefined；

  当访问对象变量中不存在的属性时会返回 undefined；

- 区别：

  - 在定义一个将要保存对象值得变量来说，可以使用 null 来初始化

- 但不建议给某个变量设置 undefined 值。字面值 undefined 主要用于比较

  - `（undefined可以用来**读**，null可以用来**读写**）`

- Number(null) return 0; Number(undefined) return NaN

### Symbol 类型（3.4）

之前没仔细了解过，原来 Symbol 还有几个 api

如果想重用符号实例，可以通过 Symbol.for()方法

```js
const a = Symbol('a');
const b = Symbol('a');
console.log(a == b); //false

const c = Symbol.for('a');
const d = Symbol.for('a');
console.log(a == b); //true

// 注：
const e = Symbol('a');
const f = Symbol.for('a');
console.log(a == b); //false
```

还可以对其进行查询

```js
const g = Symbol.for('a');
console.log(Symbol.keyFor(g)); // 'a'
const h = Symbol('a');
console.log(Symbol.keyFor(h)); // undefined

console.log(Symbol.keyFor(123)); // TypeError 123 is not a symbol
```

当 Symbol 符号作为对象的键的时候，是不会被 Object.keys 或 entries 等相关 api 遍历出来的,

可以通过 Object.getOwnPropertySymbols()等来拿到对象的 symbol 类型键值

```js
const a = Symbol('a');
const b = Symbol('b');
const obj = {};
obj[a] = '123';
obj[b] = '456';
obj['c'] = '789';
console.log(obj); // {c: '789', Symbol(a): '123', Symbol(b): '456'}
Object.keys(obj); // ['c'];
Object.values(obj); // ['789'];

Object.getOwnPropertyNames(obj); // ['c'];
Object.getOwnPropertySymbols(obj); // [Symbol(a), Symbol(b)];
Object.getOwnPropertyDescriptors(obj); // [Symbol(a), Symbol(b)];

// 或者通过Reflect.ownKeys()拿到两个类型的键
Reflect.ownKeys(obj); // ['c', Symbol(a), Symbol(b)];
```

## 四、变量、作用域与内存

### 原始值与引用值（4.1）

变量可以包含两种类型的数据，原始值和引用值，

原始值就是最简单的数据（string， number，symbol，undefined，null, boolean）

引用值则是由多个值构成的对象。

typeof 可以确认值的原始值类型，instanceof 可以用于确认值的引用类型

## 六、集合引用类型

### array.form() (6.2.1)

Array.form()除了能将类数组结构转换为数组实例外，还能进行类似 map 操作

Array.from()方法有一个可选参数 mapFn，让你可以在最后生成的数组上再执行一次 map 方法后再返回。也就是说` Array.from(obj, mapFn, thisArg) 就相当于 Array.from(obj).map(mapFn, thisArg)

## 八、对象、类与面向对象编程

### 理解对象（8.1）

对象： 一组属性的无序集合，他的每个属性或方法都有一个名称来标识；
对象的属性有两种类型，数据属性和访问器属性，ECMA 使用了一些内部特性来描述属性的特征

- 数据属性：数据属性包含一个保存数据值的位置用来读写，他有四个特性

  - [[Configurable]]: 表示当前属性是否可以通过 delete 删除以及是否可以把它改为访问器属性，默认为 true
  - [[Enumberable]]: 表示属性是否可以通过 for-in 循环返回，默认为 true
  - [[Writable]]: 表示属性值是否支持修改，默认为 true
  - [[Value]]: 包含属性实际的值，默认为 undefined。

```
let obj = {
  name: 'Xiaoming'
}
// 这就意味着obj这个对象的name属性的特性[[Value]]会被设置为'Xiaoming'

//可以通过Object.defineProperty()方法来修改属性的默认特性
// Object.defineProperty(objName, attrName, descriptor)
// objName: 要修改的对象名称；
// attrName: 要修改的对象的属性名称；
// descriptor: 要修改的描述符对象(Configurable、Enumberable、Writable、Value)。

const person = {}
// 设置只读
Object.defineProperty(person, 'name', {
  writable: false,
  value: 'Xiaoming',
})
console.log(person.name) // Xiaoming;
person.name = 'Xiaohong'
console.log(person.name) // Xiaoming;

// 设置不可删除
Object.defineProperty(person, 'name', {
  configurable: false,
})
delete person.name;
console.log(person.name) // Xiaoming;

```

- 访问器属性: 访问器属性不包含数据值，相反，他们包含一个获取函数（getter）和设置函数（setter），在读取访问器属性时，会调用获取函数，在写入访问器属性时会调用设置函数。同样他也有 4 个特性来描述他的行为:

  - [[Configurable]]: 表示当前属性是否可以通过 delete 删除以及是否可以把它改为访问器属性，默认为 true
  - [[Enumberable]]: 表示属性是否可以通过 for-in 循环返回，默认为 true
  - [get]: 获取函数，默认值为undefined，在读取属性时调用
  - [set]: 设置函数，默认值为undefined，在写入属性时调用
    【注】访问器属性是不能直接定义的，必须使用 Object.defineProperty()

```
const person {
  age_: 28,
}
Object.definedProperty(person, 'age', {
  get() {
    return this.age_ + 1;
  }
  set(newValue) {
    this.age_ = newValue * 2;
  }
})
console.log(person.age_) // 28;
console.log(person.age) // 29;
person.age = 10;
console.log(person.age_) // 20;
console.log(person.age) // 21;

// 获取函数和设置函数不一定都要定义，只设置获取函数意味着属性是只读的，只设置设置函数的属性意味着属性不能进行读取
```

- 可以通过 Object.getOwnPropertyDescriptor(objName, attrName?: string)方法来取得指定属性的属性描述符

```
const person = {name: 'Xiaoming'}
console.log(Object.getOwnDefinePropertyDescriptor(person, 'name'))
// {
//  configurable: true,
//  enumerable: true,
//  writable: true,
//  value: 'Xiaoming',
//}
```

### 对象标识及相等判定（8.1.5）

- 除了能通过 isNaN（）来判断变量是否是 NaN，还可以通过 Object.is();

```
  console.log(NaN === NaN) // false;
  console.log(isNaN(NaN)) // true;
  console.log(Object.is(NaN, NaN)) // true;

```

### 判断对象中是否包含某个属性

```
// 判断对象实例或原型是否包含某个实例
const obj = {name: 'xiaoMing'};
console.log('name' in obj) // true;
console.log('age' in obj) // false;

//判断对象实例是否包含某个实例
function Person(){};
Person.prototype.name = 'xiaoMing';
let son = new Person();
console.log(son.name) // 'xiaoMing;
console.log(son.hasOwnProperty('name')) // false;


```

