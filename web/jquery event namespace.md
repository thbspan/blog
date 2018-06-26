# jquery event namespace

平常我们在阅读代码的时候会看到类似这样的代码$('body').on('click.book.data-api', ...)，这表示什么意思呢？这就是事件的命名的命名空间。

当同一个对象同一种事件类型绑定多个事件处理函数时，一般情况下，触发某个事件，就会触发执行与之对应的所有事件处理函数；解除某种类型的事件绑定，就会解除该事件类型绑定的所有事件处理函数。

jQuery中的事件函数可以在绑定事件处理函数时，为每个事件类型定义一个或多个命名空间。使用命名空间，我们就可以只触发执行指定命名空间下的事件处理函数，或者只移除指定命名空间下绑定的事件处理函数。

当事件被触发时，event.namespace 属性返回自定义命名空间。如果没有定义则返回空字符串(**注意：如果是原生的事件，即是通过用户在页面上操作而触发的事件，不是在程序中通过trigger、triggerHandler、click等方法触发的，会返回undefined**)。

jQuery中事件类型的命名空间有点类似于类名选择器(.className)。

## 代码实例说明：

```javascript
// 1、普通的绑定click事件
$("p").on("click",function(event){
    alert("1"+event.namespace);
});
// 2、命名空间a下绑定click事件
$("p").on("click.a",function(event){
    alert("2"+event.namespace);
});
// 3、命名空间b、c下绑定click事件
$("p").on("click.b.c",function(event){
    alert("3"+event.namespace);
});
// 4、命名空间a、c下绑定click事件
$("p").on("click.a.c",function(event){
    alert("4"+event.namespace);
});

// 场景A：不限制命名空间，1/2/3/4都会触发
$("p").trigger("click");

// 场景B：限制命名空间a，2/4都会触发
$("p").trigger("click");

// 场景C：限制命名空间.a.c，这是会执行同时定义在a和c命名空间下的事件，只有4都会触发，输出4a.c
$("p").trigger("click.a.c");

// 场景D：与C类似，不过命名空间变成.c.a，也是只有4会触发，输出也是4a.c。修改事件绑定时a c的顺序输出的结果也相同，可能输出事件是按照字母顺序排列的
$("p").trigger("click.c.a");
// 场景E：移除命名空间a下的click事件，之后触发click.c。只会输出3c。4中命名空间a下的click事件移除会导致命名空间c下的事件也被移除
$("p").off( "click.a" )
$("p").trigger( "click.c" )

// 场景F：移除所以click事件
$("p").off( "click" )
```

## 使用场景

* 移除对于某一特定事件的自定义命名空间，不移除其他任何事件处理程序;
* 根据所使用的命名空间以不同的方式处理任务