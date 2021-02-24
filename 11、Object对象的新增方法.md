## 一、Object.is()
ES5 比较两个值是否相等，只有两个运算符：相等运算符（==）和严格相等运算符（===）。它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。<br>

ES6 提出Object.is()用来解决这个问题。`它用来比较两个值是否严格相等`，与严格比较运算符（===）的行为基本一致。`不同之处只有两个：一是+0不等于-0，二是NaN等于自身。`

    +0 === -0 //true
    NaN === NaN // false

    Object.is(+0, -0) // false
    Object.is(NaN, NaN) // true
## 二、Object.assign()
Object.assign()方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。

    const target = { a: 1 };

    const source1 = { b: 2 };
    const source2 = { c: 3 };

    Object.assign(target, source1, source2);
    target // {a:1, b:2, c:3}
    Object.assign(target, source1, source2,...);  //Object.assign()方法的第一个参数是目标对象，后面的参数都是源对象。
注意，如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。<br>
如果只有一个参数，Object.assign()会直接返回该参数。<br>

### 1、Object.assign()参数不是对象
（1）首参数（目标对象）不是对象的情况 <br>
如果该参数不是对象，则会先转成对象，然后返回。由于undefined和null无法转成对象，所以如果它们作为参数，就会报错。<br>
（2）非首参数（源对象）不是对象的情况 <br>
首先将这些参数都会转成对象，如果无法转成对象，就会跳过。这意味着，`如果undefined和null不在首参数，就不会报错。`<br>

* 其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了`字符串`会以数组形式，拷贝入目标对象，其他值都不会产生效果。

        const v1 = 'abc';
        const v2 = true;
        const v3 = 10;

        const obj = Object.assign({}, v1, v2, v3);
        console.log(obj); // { "0": "a", "1": "b", "2": "c" }
上面代码中，只有字符串合入目标对象（以字符数组的形式），数值和布尔值都会被忽略。这是因为只有字符串的包装对象，会产生可枚举属性。

    Object(true) // {[[PrimitiveValue]]: true}
    Object(10)  //  {[[PrimitiveValue]]: 10}
    Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
上面代码中，布尔值、数值、字符串分别转成对应的包装对象，可以看到它们的原始值都在包装对象的内部属性`[[PrimitiveValue]]`上面，`这个属性是不会被Object.assign()拷贝的`。`只有字符串的包装对象，会产生可枚举的实义属性，那些属性则会被拷贝。` <br>

Object.assign()拷贝的属性是有限制的，只拷贝源对象的自身属性（不拷贝继承属性），（属性名为 Symbol 值的属性，也会被拷贝）也不拷贝不可枚举的属性（enumerable: false）。<br>

### 2、注意点<br>
（1）浅拷贝 <br>
Object.assign()方法实行的是浅拷贝，而不是深拷贝。如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的`引用`。 <br>

（2）同名属性的替换<br>
对于这种嵌套的对象，一旦遇到同名属性，Object.assign()的处理方法是替换（覆盖） <br>

（3）数组的处理<br>
Object.assign()可以用来处理数组，但是会把数组视为对象。<br>

    Object.assign([1, 2, 3], [4, 5])
    // [4, 5, 3]
Object.assign()把数组视为属性名为 0、1、2 的对象，因此源数组的 0 号属性4覆盖了目标数组的 0 号属性1。<br>

（4）取值函数的处理<br>
Object.assign()只能进行值的复制，如果要复制的值是一个取值函数，那么将求值（得到返回值）后再复制。
### 3、常见用途
（1）为对象添加属性<br>
（2）为对象添加方法<br>
（3）克隆对象<br>
不过，采用这种方法克隆，只能克隆原始对象自身的值，不能克隆它继承的值。如果想要保持继承链，可以采用下面的代码。

    function clone(origin) {
      let originProto = Object.getPrototypeOf(origin);
      return Object.assign(Object.create(originProto), origin);
    }
（4）合并多个对象<br>
（5）为属性指定默认值<br>
注意默认值为对象时可能存在的浅拷贝问题
## 三、Object.getOwnPropertyDescriptors()
ES5 的Object.getOwnPropertyDescriptor()方法会返回某个对象属性的描述对象（descriptor）。ES2017 引入了Object.getOwnPropertyDescriptors()方法，返回指定对象`所有`自身属性（非继承属性）的描述对象，如果没有任何自身属性，则返回空对象

    Object.getOwnPropertyDescriptor(obj, prop);  //prop是目标对象内属性名称
    Object. getOwnPropertyDescriptors(obj);
用途：<br>
（1）解决Object.assign()无法正确拷贝get属性和set属性的问题。<br>
Object.getOwnPropertyDescriptors()方法配合Object.defineProperties()方法，就可以实现正确拷贝。<br>

    Object.defineProperties()方法是创建/修改/获取属性的方法，该方法直接在一个对象上定义一个或多个新的属性或修改现有属性，并返回该对象

    Object.defineProperties(obj, props)
    obj: 将要被添加属性或修改属性的对象
    props: 该对象的一个或多个键值对（对象形式），定义了将要为对象添加或修改的属性的具体配置

    const source = {
      set foo(value) {
        console.log(value);
      }
    };

    const target2 = {};
    Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));   //将source所有自身属性的描述对象添加到target2到对象中
    Object.getOwnPropertyDescriptor(target2, 'foo')  //返回target2对象的foo属性的描述对象
    // { get: undefined,
    //   set: [Function: set foo],
    //   enumerable: true,
    //   configurable: true }

（2）配合Object.create()方法，将对象属性克隆到一个新对象（浅拷贝）

    Object.create(proto, [ propertiesObject ])  创建一个拥有指定原型的对象，并为该原型对象设置属性

    const clone = Object.create(Object.getPrototypeOf(obj),  
      Object.getOwnPropertyDescriptors(obj));  
    //把obj对象的所有自身属性的描述对象添加到obj的原型对象上。这样clone就拥有了obj所有自身和其继承的属性了

    // 或者

    const shallowClone = (obj) => Object.create(
      Object.getPrototypeOf(obj),
      Object.getOwnPropertyDescriptors(obj)
    );
（3）实现一个对象继承另一个对象<br>
以前，继承另一个对象，常常写成下面这样

    const obj = {
      __proto__: prot,
      foo: 123,
    };
如果不用__proto__，上面代码就要改成下面这样。

    const obj = Object.create(prot);
    obj.foo = 123;

    // 或者

    const obj = Object.assign(
      Object.create(prot),
      {
        foo: 123,
      }
    );
有了Object.getOwnPropertyDescriptors()，我们就有了另一种写法。

    const obj = Object.create(
      prot,
      Object.getOwnPropertyDescriptors({
        foo: 123,
      })
    );
（4）实现 Mixin（混入）模式。

    let mix = (object) => ({
      with: (...mixins) => mixins.reduce(
        (c, mixin) => Object.create(
          c, Object.getOwnPropertyDescriptors(mixin)
        ), object)
    });

    // multiple mixins example
    let a = {a: 'a'};
    let b = {b: 'b'};
    let c = {c: 'c'};
    let d = mix(c).with(a, b);

    d.c // "c"
    d.b // "b"
    d.a // "a"
上面代码返回一个新的对象d，代表了对象a和b被混入了对象c的操作。
## 四、__proto__属性，Object.setPrototypeOf()，Object.getPrototypeOf()
__proto__属性存储了当前对象的原型对象<br>

    Object.prototype.__proto__  //读取原型对象
由于这是内部属性，所以最好不要直接调用它。可通过以下方法对原型对象进行操作：<br>
Object.setPrototypeOf()（写操作）、Object.getPrototypeOf()（读操作）、Object.create()（生成操作）。<br>

Object.setPrototypeOf(obj, proto);   //将proto对象设为obj对象的原型，并返回第一个参数obj对象<br>
如果第一个参数不是对象，会自动转为对象。但是由于返回的还是第一个参数，所以这个操作不会产生任何效果。<br>
由于undefined和null无法转为对象，所以如果第一个参数是undefined或null，就会报错。<br>

Object.getPrototypeOf(obj);  //读取obj对象的原型对象 <br>
如果参数不是对象，会被自动转为对象。如果参数是undefined或null，它们无法转为对象，所以会报错。
## 五、Object.keys()，Object.values()，Object.entries()
（1）Object.keys()  返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的`键名`。<br>

（2）Object.values()  返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的`键值`。<br>
Object.values方法会过滤属性名为 Symbol 值的属性。<br>
如果Object.values方法的参数是一个字符串，会返回各个字符组成的一个数组。 <br>
如果参数不是对象，Object.values会先将其转为对象。由于数值和布尔值的包装对象，都不会为实例添加非继承的属性。所以，Object.values会返回空数组。<br>

（3）Object.entries()方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的`键值对数组`。

    const obj = { foo: 'bar', baz: 42 };
    Object.entries(obj)
    // [ ["foo", "bar"], ["baz", 42] ]
除了返回值不一样，该方法的行为与Object.values基本一致。<br>

Object.entries的基本用途是遍历对象的属性。<br>
Object.entries方法的另一个用处是，将对象转为真正的Map结构。

    const obj = { foo: 'bar', baz: 42 };
    const map = new Map(Object.entries(obj));
    map // Map { foo: "bar", baz: 42 }
六、Object.fromEntries()
Object.fromEntries()方法是Object.entries()的逆操作，用于将一个键值对数组转为对象。

    Object.fromEntries([
      ['foo', 'bar'],
      ['baz', 42]
    ])
    // { foo: "bar", baz: 42 }

用途：<br>
（1）由于方法的主要目的，是将键值对的数据结构还原为对象，因此特别适合将 Map 结构转为对象。

    // 例一
    const entries = new Map([
      ['foo', 'bar'],
      ['baz', 42]
    ]);

    Object.fromEntries(entries)
    // { foo: "bar", baz: 42 }

    // 例二
    const map = new Map().set('foo', true).set('bar', false);
    Object.fromEntries(map)
    // { foo: true, bar: false }
（2）配合URLSearchParams对象，将查询字符串转为对象。

    Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'))
    // { foo: "bar", baz: "qux" }
