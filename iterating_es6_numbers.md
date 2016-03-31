# Iterating ES6 Numbers 

ES6(又称为“ES2015”)提出了迭代器的概念，使用迭代器接口可以遍历数据集合中的每一个元素，这也是在其他主流语言中存在的让人期待已久的接口。

想了解更多遍历器相关知识，可以阅读我的[《你不知道的JS》](http://youdontknowjs.com/)图书系列中的[《异步&性能》](https://github.com/getify/You-Dont-Know-JS/tree/master/async%20%26%20performance#you-dont-know-js-async--performance)[第4章](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20&%20performance/ch4.md#chapter-4-generators)和[《ES6及以上》第2章](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20&%20beyond/toc.md#you-dont-know-js-es6--beyond)

下面这段代码展示了如何使用ES6迭代器：

    var it = getMeAnIterator();

    it.next();
    it.next( 42 );
    ..
每次调用`next(..)`方法，都会返回如下格式的对象：
    
    { done: true/false, value: .. }
JS中有几种内置数据类型是可以迭代的，常用的有Array和String，这意味着你可以从这种类型的对象中获取迭代器去循环遍历对象中的值。

为了帮助完成这种数据结构的遍历，ES6创造了`for..of`循环命令，使用`for..of`遍历对象代码如下：

    var a = [1, "hello", 42, "world!"];

    for (var i of a) {
        console.log( i );
    }
    // 1
    // "hello"
    // 42
    // "world!"
  
    var s = "abc";
    for (var c of s) {
        console.log( c );
    }
    // "a"
    // "b"
    // "c"
酷吧？  
**注意：**你可能认为对象也会像数组那样有个内置迭代器，然而并非如此。其中的原因我们不在这里细说，你可以在下一章节中了解到相关内容。

### 自定义迭代器
除了拥有内置迭代器的几种数据结构外，也可以自定义迭代器。例如，我们可以构造一个对象，遍历时返回从0到10的偶数

    var evens = {
        [Symbol.iterator]: function(){
            var i;
        
            return {
                next: function() {
                    if (i == null) {
                        i = 0;
                        return { value: 0, done: false };    
                    }
                    else if (i <= 8) {
                        i += 2;
                        return { value: i, done: false };
                    }
                    else {
                        return { done: true };
                    }
                }    
            };
        }
    };

    for (var i of evens) {
        console.log( i );
    }
    // 0
    // 2
    // 4
    // 6
    // 8
    // 10
这段代码中，`Symbol.iterator`是ES6中新增的Symbol类型，这种数据类型非常特殊，他不与任何值相等，是唯一的，通常用于标识特殊属性，避免属性名冲突。在这个例子中`Symbol.iterator`不是迭代器本身，而是语言预定义的内置迭代器工厂方法，通过调用这个方法产生一个迭代器实例。例如，数组的内置迭代器工厂可以被如下方式访问（甚至覆盖）：

    var a = [1,2,3];

    var it = a[Symbol.iterator]();

    it.next();  // { value: 1, done: false }
    ..
所以在上面那个例子中，我们用`[Symbol.iterator]`做为属性名声明了一个迭代器工厂,而不是用`foo`之类普通的属性名.中括号用在对象直接量的属性位置，是ES6另一个新增的语法，叫做计算属性名，你可以在中括号中间放置任何JS表达式，计算结果将会被用作属性名。例如：

    var a = "fo", b = "o";

    var o = {
       [a + b]: 42
    };

    o.foo;  // 42

回到evens的例子中，我们定义了一个方法，调用时返回迭代器，该迭代器对象只有一个`next(..)`方法，`next(..)`方法每次调用时按照要求返回`{ value: .. , done: .. }`格式的数据

**注意：**严格来说，内置迭代器拥有`next(..)`和`return(..)`方法。另外生成迭代器要用`throw(..)`方法，抛开这些问题不管，我们已经成功构造了一个自定义的迭代器，下面就该使用了。

`for..of`循环会自动查找对象的迭代值（evens例子中的偶数），并且检查对象是否拥有一个`Symbol.iterator`属性的迭代器，如果有，则会调用这个迭代方法，返回值被用到`for..of`循环中处理。
`for..of`会一直查找直到迭代器返回的数据格式中包含`done:true`。最后，每次成功的循环迭代返回的数据中的value值都会被赋值到循环索引变量中（evens示例中是i）。

### 迭代数字
现在让我们用新get的技能娱乐一下吧。如果可以让JS中的所有数字变得可迭代，将会是什么样呢？我首先想到的是可以从0递增遍历到当前数字（负数的话要从0递减）。如果愿意动手做的话，甚至可以以每步大于1的方式跳过一些数字遍历，使用方法如下：

    for (var i of 7) {   // much nicer than:   for (var i=0; i<= 7; i++) {
        console.log( i );
    }
手动控制迭代：

    var it = 8[Symbol.iterator]( 2 ); // default to stepping by 2

    it.next();  // { value: 0, done: false }
    it.next();  // { value: 2, done: false }

    // now let's step by 4 (2 * 2)
    it.next(2); // { value: 6, done: false }

    it.next();  // { value: 8: done: false }
    it.next();  // { done: true }

`for(var i of 7)`比实现同样作用的典型for循环要简洁很多。

下面就来揭晓如何实现这样的小花招，给Numer.prototype添加一个自定义的迭代器：

    if (!Number.prototype[Symbol.iterator]) {
        Number.prototype[Symbol.iterator] = function(inc){
            var i, done = false, top = +this;
        
            // iterate positively or negatively?
            inc = Math.abs(Number(inc) || 1) * (top < 0 ? -1 : 1);
                
            return {
                // make the iterator itself an iterable!
                [Symbol.iterator]: function(){ return this; },

                next: function(step) {
                    // increment by `step` (default: 1)
                    step = Math.abs(Number(step) || 1);
                
                    if (!done) {
                        // initial iteration always 0
                        if (i == null) {
                            i = 0;
                        }
                        // iterating positively
                        else if (top >= 0) {
                            i = Math.min(top,i + (inc * step));
                        }
                        // iterating negatively
                        else {
                            i = Math.max(top,i + (inc * step));
                        }
                    
                        // done after this iteration?
                        if (i == top) done = true;

                        return { value: i, done: false };
                    }
                    else {
                        return { done: true };
                    }
                }
            };        
        };
    }
**更新：**感谢ziyunfei的评论，我们已经让迭代器本身可迭代，这样就可以在任何JS执行迭代的地方手动实例化迭代器，比如上面的Number迭代器可以写成`for (var i of 8[Symbol.iterator](2)) { .. }`,酷吧？就像我们之前讨论的自定义迭代器一样简单明了。

**注意：**不要忘了ES6特性需要在支持ES6的浏览器或者使用转码器（如[6to5](http://6to5.org/)）转换才能执行。如果你的浏览器不支持这篇博客中的一些特性，可以使用在线[REPL demo](http://6to5.org/repl)

### 数字范围
**更新**再次感谢ziyunfei的评论，我添加了这段内容。ES6新增的`...`(三个点)运算符，根据上下文环境不同有几种不同的用法。我只覆盖了其中一种，在可迭代的值前使用，将自动展开所有的迭代结果值，所以我们可以利用这个特性达到取出数值范围的效果。

    var range1 = [ ...18 ];
    range1; // [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18]

    var range2 = [ ...18[Symbol.iterator](3) ];
    range2; // [0,3,6,9,12,15,18]
### 总结
ES6迭代器和for..of循环为遍历一组数值提供了的便捷的语法支持，使用...可以让我们得到获取数值范围的语法支持

不是所有JS数值类型都有内置迭代器，比如object和number。但是我们可以通过自定义实现。这个技巧如此巧妙以至于你可能会疑惑为什么ES6不包含这个number迭代器。

大家一起来参与讨论吧，还有很多其他的好玩的自定义迭代器。