## <a name='table_of_contents'></a>Things discussed in this documentation:
1.  [Prototypical Inheritance in JS](#prototypical_inheritance)
2.  [Scope in JS](#scope_in_js)
3.  [Function Declaration and Function Expression](#function_dec_and_expression)
4.  [Try Catch in JS](#try_catch)
5.  [Scope model in JS](#scope_model_in_js)
6.  [IIFE Pattern](#iife_pattern)
7.  [Hoisting](#hoisting)
8.  [This keyword](#this_keyword)
9.  [Closure](#closure)
10. [Classic Module Pattern](#classic_module_pattern)
11. [Modified Module Pattern](#modified_module_pattern)
12. [Modern Module Pattern](#modern_module_pattern)
13. [Object Orientation](#object_orientation)
    1. [Prototype](#prototype)
    2. [Inheritance](#inheritance)
    3. [OLOO](#oloo)
    4. [Object Create method](#object_create)
14. [Async Pattern](#async_patterns)
    1. [Callbacks](#callbacks)
    2. [Generators](#generators)
    3. [Promises](#promises)

## <a name='prototypical_inheritance'></a> Prototypical Inheritance in JS

  >In JavaScript when one object inherits from another prototypically, you are able
  to access all the properties and methods from the parent object.

```js
    var obj1 = {  
      someProp: 'obj1 property!',
      someMethod: function () {
        alert('obj1 method!');
      }
    };
    var obj2 = Object.create(obj1);  
    obj2.someProp = 'obj2 property!';
```  

You might think from the above code that `someProp` was
**changed** from `obj1 property!` to `obj2 property!` when obj2
was instantiated. However, you can not change properties on an
`object's prototype` like that. All that code did was `create a property` called `someProp` on `obj2` that `masks` the value
of the `underlying someProp on obj1`. If you run `delete obj2.someProp` then `someProp won't be gone`, it will 
`revert to showing` the value of `obj1.someProp`. This is the
way **nested scopes** work. Setting a property on a `child scope`
does not `change the property` with the same name on the 
`parent scope`; __it merely hides it__.

[Back to table of contents](#table_of_contents)

## <a name='scope_in_js'></a> Scope in JS:
There are only two scopes:
1. Function scope
2. Global scope

Function are the atomic unit of the `scope`.
When we define something like below:

**Compilation phase**:
```js
    var foo = "bar"; /* declaration */ 
    /* 
        While compiling this one statement gets broken into two
            1. the declaration part using var
            2. the assignment part 
    */

    function bar() { /* declaration */
        var foo = "bar"; /* declaration */
    }

    function baz(foo) { /* declaration of function as well as foo */
        foo = "bam";
        bam = "yay";
    }
```

Things to note about the above code is:
1. JS do JIT(Just in time) compilation, which means function are
 not compiled when they are defined but they gets 
 compiled and called at the time of call.
2. Everything is scoped.

So, the very first line has a scope of `global`,
while `foo` inside the `bar` function has the scope of `bar`.
When the js compiles the `baz` function, it treats the 
`argument as a declaration`.
So `foo` in the argument will be treated as a local variable.
And the `scope` will be of `baz`.

**Execution phase**:  
Now we have to note that there will be no `var` in the execution
phase. So the first line will be:
>foo = "bar" without any `var`.

So even if we define `var foo` 100 times, while execution it
will be ignored. So defining `var foo` in somewhere in our
program has no impact on the rest as that will be ignored.

**Compiler terminologies**:
>`lhs` : left hand side  
>`rhs` : right hand side of an assignment.

In our language its the `=` sign.
So in first line:
1. `foo` will be our `lhs` reference
2. `bar` will be our rhs reference
    
But `lhs` and `rhs` can occur without assignment operator.
So to need to understand them in some other way which is:
>`lhs` : **the target**  
 `rhs` : **the source**

So in case of function `argument` there is an implicit assignment.
When we execute the function `bar` we will ask the `scope of bar`
that whether it knows about the `foo` which happens to be the
`lhs`.

When we execute the baz, we will ask scope of `baz`, whether it
knows about an `identifier` named `foo` which happens to be
the `lhs`.  
So it will say that: **yes**, `its in my declaration`.  

But when we execute the line: 
>bam = `yay`

We will ask `baz` that whether it heard about an `lhs` identifier
named `bam`. It will say: **no**.

Now where do we go?
>`We go outer one level`. so it will be our `global` scope in
 this case.

So we will ask our `global` scope, whether it knows any
identifier named `bam` which happens to be the `lhs`.

And it will say: `yes`, i have just created it for you.
The `global` scope will create `bam` for us because we have an
`lhs` reference.

>Note: This thing is in `non strict` mode.  
So this assignment will create a `global variable`.  
And that is why we should not declare a variable without `var`.

And that's why we get the idea of `leakage of global variables`.
But when we will be in `strict mode` the global scope will say
it `does not` know.

While passing `foo` in the function `baz`, even if we try to
redeclare `foo` using `var foo`, the compiler will ignore it as
it has `foo` already.

**Note**:
> In JS `undeclared` and `undefined` are two different
  words.  
  &nbsp;&nbsp;  `undeclared`: when we do not declare the identifier.  
  &nbsp;&nbsp;  `undefined` : when we declare but it contains empty value.  
  
>`undefined` is more like `uninitialised`.  
 `undefined` is an actual value.    

Example:
```js
    var foo = "bar";
    function bar() {
      var foo = "baz";
      function baz(foo) {
        foo = "bam";
        bam = "yay";
      }
      baz();
    }
    bar();
    foo;
    bam;
    baz();
```

While compiling we will say:
>hey `global scope` I have a declaration for variable called `foo`.

For compiling the `baz` function it will ask
the scope of `bar` to register the function `baz`.

While executing when we will go to the `bar()`.
We will ask: hey global scope do you have a `rhs`
reference `bar`.

**Note**: 
>bar() will be an `rhs` not `lhs` because
there is no assignment going on at this point of time.

So it will say `yes` and it will return the `function object bar`.  
We have `()` after the `bar`, so bar will have function object
and `()` will be used for calling the function.

**Note**: 
>In case we ask some `rhs` value which is not 
declared in the scope, the behavior is different.  
The global scope does not create one for us if
its a `rhs`. And it will throw a reference error.  
This is common to `restrict` and `unrestrict` mode.

[Back to table of contents](#table_of_contents)

## <a name='function_dec_and_expression'></a>Function Declaration and Function Expression:
```js
    var foo = function bar() {
        var foo = "baz";
        function baz(foo) {
            foo = bar;
            foo;
        }
        baz();
    };

    foo();
    bar();
```

>If the `function` keyword is the first word in the 
 statement then its a `function declaration`.
 Otherwise it will be a `function expressions`.

In our case it is function `expression` rather than `declaration`.

Most of the time `function expressions` are `anonymous function`,
but in our case it is `named anonymous` function.

As the first line in our code is `function expression`, `bar` will
not get defined in the `outer scope`.
  
In case of `function expression`, the name of the function is
enclosed in its `own scope`.

So the name `bar` exists throughout the function.  
So calling `foo = bar` is fine. But `bar` does not
exists in the global scope. So calling `bar()` at the last line
will raise `reference error`.

Why we should use `named anonymous function expression`:
>When we use anonymous function expression there are three
 downside:
>1. When we don't use named anonymous function expression we
 `dont have any way to refer` to ourselves. This property is
  required when we do recursion, or when we bind some event to
  our function. (name is necessary) (many people think that
  `this` keyword is a reference to itself but it is not.)
>2. `Anonymous function expression` pose problem while debugging.
  So giving a name is always helpful.
>3. We don't need to look outside to know what is being done by
 the function.
 ex:   
 when we give name as `handler` it is evident that this
 function is handling something.

[Back to table of contents](#table_of_contents)

## <a name='try_catch'></a> Try catch in JS:
In Es3, `try` `catch` was added, and according to the 
specification, `catch` has `block` scope.

Meaning, the variable which you will declare in the 
`catch` clause will only be accessible in the `catch` clause
and not `outside`.
```js
    var foo;
    try {
      foo.length;
    }

    catch (err) {
      console.log(err);
    }

    console.log(err); // ReferenceError.
```
So here `err` cant be accessed from outside.

[Back to table of contents](#table_of_contents)

##<a name='scope_model_in_js'></a> Scope Models in JS:
There are two:
1. Lexical scope which we were talking till now. (lex refers to
 the `lexing` that occurs in the compiler) (lexical scope means
 `compile time scope`)    
2. Dynamic scope (not present in js)(`run time scope`).

[Back to table of contents](#table_of_contents)

## <a name='iife_pattern'></a> IIFE Pattern (immediately invoked function expression):
```js
    var foo = "foo";
    (function() {
      var foo = "foo2";
      console.log(foo); // `foo2`
    })();

    console.log(foo); // `foo`
```

When we want to keep some statements in `segregated scope`, we use
this pattern. In this pattern an `anonymous function expression`
has been used and is executed right away.

We can have `named function expression` rather than `anonymous`
function expression. But that thing will give out the `name`.
And we wanted to hide our code in some other scope which will
not gets accessed from outside.

[Link to blog](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)

[Back to table of contents](#table_of_contents)

##<a name='hoisting'></a> Hoisting:
Its not a proper word in the specs, its only being used to
define the js behaviors.
Consider the below code:
```js
  a;   
  b;
  var a = b;
  var b = 2;
  b;
  a;
```
What will happen if we execute this code?  
Before executing there will be compilation. During which the
compiler will find the declaration first, so the above code will
transform into:
```js
    var a;
    var b;
    a;
    b;
    a = b;
    b = 2;
    b;
    a;
```

In the above code the compiler moves the declaration
of `a` and `b` to the top. They were treated first.  
`This movement of declaration to the top` is called `hoisting`.

So the compile phase will be line 1 and 2.
All the function and variable declaration will get
hoisted.  
**Note**: 
>the `function expression` does `not` get `hoisted`.

So the below code 
```js
  var a = b();
  var c = d();
  a;
  c;

  function b() {
    return c;
  }

  var d = function() {
    return b();
  };
```
Will be converted to
```js
  function b() {
    return c;
  }
  var a;
  var c;
  var d;
  a = b();
  c = d();
  a;
  c;
  d = function() {
    return b();
  };
```
**Note**: 
>Functions are `hoisted` before variables.

Consider the below code:
```js
    foo();
    var foo = 2;
    
    function foo() {
      console.log("bar");
    }
    
    function foo() {
      console.log("foo");
    }
```

In this case first function `foo` containing `bar` will be
`hoisted`. Then the function `foo` containing `foo` will be
hoisted which will override the previous hoisted value.

Then the variable `foo` gets hoisted and it will be ignored as
there is already a variable called `foo` and it holds the last
function.

**Note**:
>**Mutual recursion**: Two function calling each other till a 
   terminating condition is reached is 
   called `Mutually recursive` function.  
   And mutual recursion is not possible in a language where
   there is not `hoisting`.  
   `header files in c language` are manual hoisting. we are
   putting the declaration part on the top of everything.  
   **Js automatically do hoisting**.

[Back to table of contents](#table_of_contents)

##<a name='this_keyword'></a>The `this` keyword:
Every function, while executing, has a reference to its current
execution context, called `this`.
Execution context include far more than `this` keyword,
but we are only interested in `this`.
`Execution context` means `where the function has been called
and how the function has been called`.

There are four rules for how the `this` keyword is bound.
And it all depends on `call sight`(It is the place in code where the `function is executed`).

**Four rules**: 
1. `Default binding rule` (4th according to preference)  
    ex:
    ```js
      function foo() {
        console.log(this.bar);
      }
    
      var bar = "bar1";
      var o2 = {bar: "bar2", foo: foo};
      var o3 = {bar: "bar3", foo: foo};
    
      foo(); // here there is only reference to the function
            // and nothing else.
            // In these situation, the default binding rule
            // applies. 
            // This is also true with the IIFIs.
      o2.foo();
      o3.foo();
    ```

    >Default rule says:
    1. If you are in strict mode, default to `this` keyword is
     `undefined` value.
    2. If not, default to `this` keyword is global object.

    **Note**: 
    > In js everything is a reference to an object. In the above
     code we have two reference to the `foo`.  
    >1. Our `global` variable is referencing the `foo`.
    >2. o2.foo is also referencing the `foo`.
    
2. `Implicit binding rule`: ( 3rd according to preference )
    Consider the above code:
    >we have o2 and o3 which have same function `foo`.
     So we have an `implicit reference` for `foo`.  
     If o2 will not have the `bar` property, `this.bar` will not
     refer to the `global bar`.
    
    **Note**: 
    >Binding confusion:
    ```js
        function foo() {
          var bar = "bar1";
          baz();
        }
    
        function baz() {
          console.log(this.bar)
        }
    
        var bar = "bar2";
        foo();
    ```
    >This code is fake but the concept is real.
      Here the function `baz` is some `third party`
      function on which the user does not have any
      control but user knows that this function is 
      using `this.bar` somewhere in the code, on the 
      other hand user have another function `foo` on
      which user have control. Now user have defined
      `bar` locally and is trying to call `bar()` in 
      a hope of referencing the `local variable bar`.
      
    >Note: `It is impossible to create a crossover between the
     lexical environment and the this mechanism`.
     They are just two fundamentally different
     mechanism and they don't crossover.

    The above code was not doing what was expected from it.  
    The incorrect solution:
    ```js
        function foo() {
          var bar = "bar1";
          this.baz = baz;
          this.baz();
        }
    
        function baz() {
          console.log(this.bar);
        }
    
        var bar = "bar2";
        foo();
    ```
    How the above code is wrong:
    >`this` reference gets set by the `sight of call`.
      In our code we need to find what is being referenced
      by the `this` keyword.
          
      As we can see that the function `foo` is 
      being called from the global scope, so 
      `this` will point to the global object as
      we are in not strict mode.
      So when we are saying:
      >this.baz = baz in function `foo`.  
       We are actually saying global.baz = baz
       but `global.baz` is already there.
    
      when we are calling the `baz` function in
      our `foo` function, this.baz(): here the `implicit rule`
      applies as we are calling `object.function()` but the
      `object` is still `global` object, so it will be `global.baz()`.

3. `Explicit scope`:
    ```js
        function foo() {
          console.log(this.bar);
        }

        var bar = "bar1";
        var obj = {bar: "bar2"};

        foo();            // bar1
        foo.call(obj);    // bar2
    ```
    Explicit binding say that:
    >If we use `.call()` or `.apply()` at the call sight, 
     both of these utilities take their `first parameter` 
     as `this` binding.

    Problem with `this` keyword generality:
    > In case of a controller, when we have `this` 
      pointing to the controller object, we can do all 
      our work by just using `controller.method`.
      
    >But when we do an ajax call, the things get a bit 
      different. We say in the callback I want to call my 
      controller method :  
      we pass an reference to the controller method.
      But when it gets called the `this` binding gets 
      `wrong`, it will `fall back` to the `global`.

    >Or say we have attached it to an event handler 
    like a `click handler` on a button. When I click 
    the button, I want my controller method to 
    invoked but we will get frustrated because `this`
    binding becomes the `button` itself rather than 
    my `controller object`.

    **solution**:
    >`hard binding`: It is used to fixate the `this` keyword
     reference.  
     ex: 
     ```js
        function foo() {
          console.log(this.bar);
        }

        var obj = {bar: "bar"};
        var obj2 = {bar: "bar2"};

        var orig = foo;
        foo = function() {orig.call(obj);};

        foo();    // `bar`
        foo.call(obj2);  // ???
     ```

     In the above code, everything is same apart from
      declaration of `orig` variable and assigning it 
      'foo'. Now in the next line, we are overriding the 
      foo with a `function expression`, and `forcing` the
      call to `foo` to the original `foo` but fixing the 
      `this` reference to the `obj`.

     So both the function call will print `bar`.

     **Making an binding utility**:
     ```js
      function bind(fn, o) {
        return function() {
          fn.call(o);
        };
      }
      function foo() {
        console.log(this.bar);
      }

      var obj = {bar: "bar"};
      var obj2 = {bar: "bar2"};

      foo = bind(foo, obj);

      foo();           // `bar`
      foo.call(obj2);  
     ```

      This code is same as the above one.
      But it does not have any variable in 
      `global scope` whose reference can be 
      over written.
      But in this code too we are creating one 
      `global utility`.

      Also the utility in this code does not have
      return statement(we cant return anything from the func)
      And we can not pass arguments too.

      **solution**:  
      putting the utility on the prototype of function
      itself.
     ```js
        // we are calling our utility `bind2` temporarily.
        if (!Function.prototype.bind2) {
          Function.prototype.bind2 = 
              function(o) {
                var fn = this; // the function!
                // when we will look at the call sight
                // we will find that the utility is 
                // a function, so 'this' will apply to it.
                // also we are calling it using 'implicit'
                // binding of 'foo'. So 'this' will point to
                // 'foo'.
                return function() {
                  return fn.apply(o, arguments);
                };
              };
        }

        function foo(baz) {
          console.log(this.bar + " " + baz);
        }

        var obj = {bar: "bar"};
        foo = foo.bind2(obj);

        foo("baz")    // `bar baz`

     ```
     
      In this code the utility has a return statement so
      it is capable of returning some values, also
      we will be able to pass arguments in this utility.

      It is so common that the JavaScript has a `bind()`
      function on the function `prototype`.
      So we don't need our `bind2` utility. Its built-in.
      (In ECMAScript 5)

      When we go to the MDN page for binding, they have a 
      `polyfills`(they are to provide support for older browser).
      So we can use the `bind` utility in older browser too.


4. `new keyword`:
   >Please set aside any conception about the 'new' keyword
    In other language `new` keyword is used for instantiating 
    the classes but
   >1. JavaScript does not have `classes`
   >2. `new` keyword has nothing to do with the `instantiation`
   ```js
    function foo() {
      this.baz = 'baz';
      console.log(this.bar + ' ' + baz);  // undefined undefined
    }

    var bar = 'bar';
    var baz = new foo();
   ```
   
   When we put the `new` keyword in front of any function
   call, it magically turns that `function call` into a
   `constructor` call.

   When we put the `new` keyword in front of any function
   call, it's going to do `4 things`:
   1. A `brand new` empty `object` will be created.
   2. The new empty `object` gets `linked` to a different
      object.
   3. The `brand new` empty object get `bound` as the `this` 
      keyword for the purposes of that function call.
   4. If that function does not otherwise return anything.
      Then it will implicitily insert, between `line 3` and `half
      of the above code`, a `return this`.
      That is the `brand new` object get returned for the 
      purposes of the call.

   So in the above code, when we call `foo` function with
      the `new` keyword, an object as the `this` reference
      will get returned.

      So we can have one variable on `this` as `ba`, but
      when we try to access `this.bar`, as there is nothing
      called `bar` on `this`, therefore, `undefined` will
      gets printed.

      And at the moment, `baz` variable exists but it does not
      have any value so we will print `undefined` again.
      
      But final thing is that, there is a implicit `return
      this`, so the newly  created object gets assigned 
      to our `baz` variable.
      So if we try `baz.baz` we will get the string value 'baz'.

   How `this` binding works:
   >Ask these questions:
   >1. Was the function called with 'new' keyword ?
   If so, use that object.
   Which means the `new` keyword is able to  override 
   any of the other rules as it is most `precedent` of the
   rules.
   >2. Was the function called with `call` or `apply`
   specifying an explicit `this`?
   If so, use that object.
   >3. Was the function called via a containing/ owning
    object(context) ?
   >4. Default: global object (except strict mode)

   >`hard bound` functions are a variation of explicit binding
    rule.

    So, what will be the precedence of `hard binding`?
    >Ans: At no 2, which means the `new` keyword is able to 
      override the `hard binding`.

[Back to table of contents](#table_of_contents)

##<a name='closure'></a> Closure:
>Closure is when a function `remembers` its `lexical scope`
even when the function is executed `outside that lexical` scope.

Ex:
```js
  function foo() {
    var bar = "bar";

    function baz() {
      console.log(bar);
    }

    bam(baz);
  }

  function bam(baz) {
    baz();        // `bar`
  }

  foo();
```

**Note**:
>If we keep a reference to an object, that does not get `garbage` collected.
Similarly until we have at-least one function referencing
    the `scope` object, through `closure`, `scope` does not 
    gets garbage collected.

Prob:
>consider the below code:
```js
for (var i = 1; i <= 5; i++) {
  setTimeout(function() {
    console.log("i: " + i);
  }, 1 * 1000);
}
```

By running this example we will get the below result:
  i: 6  
  i: 6  
  i: 6  
  i: 6  
  i: 6  
  i: 6  

But why?
>Because through `closure` the function inside `setTimeout`
 is using the the global variable `i`.
 and as we have live copy of the `i` variable in each 
 function call, every `i` value will get updated with 
 the last value.

How to solve it ?

1. Using `IIFI` pattern: we need to use the `IIFE` pattern.      
   Ex:
   
    ```js
      for (var i=1; i<= 5; i++) {
        (function(i){
          setTimeout(function(){
            console.log('i: ' + i);
          }, i * 1000);
        })(i);
      }
    ```
    so now each of the `setTimeout` function will have
    iteration scope rather than `global` scope.

2. Using `let` keyword:
    ```js
        for (let i=1; i<5; i++) {
          setTimeout(function(){
            console.log("i: " + i);
          },i*1000);
        }
    ```
    what `let` keyword does is, it rebinds `i` for 
    each `iteration` of `for` loop.

[Back to table of contents](#table_of_contents)

##<a name='classic_module_pattern'></a>Classic module pattern:
>It has two characteristics:
>1. There must be an outer wrapping function that
        gets executed(Does not have to be a IIFE, but it
          does have to an outer function that gets
          executed).
>2. There must be one or more functions that get
        returned from that function call. So one or more
        inner functions that have the `closure` over the
        inner private scope.
        
Ex:
```js
var foo = (function() {
  var o = {bar: 'bar'};

  return {
    bar: function() { //these stuffs are like
                      // private members of a 
                      // module.
                      // And the obj `bar` is like
                      // an public API.
                      
                      // There are one or more methods
                      // on the API, that have the
                      // special privilleged 'closure'
                      // capability that access the 
                      // 'internal' state, and that
                      // makes a module. 
      console.log(o.bar);
    }
  };
})();

foo.bar();
```
So all the function is `hidden` from the outside world
and we get choose what we return on our public `API`.

>This pattern is useful in implementing `encapsulation`. This is the idea of hiding private implementation details

[Back to table of contents](#table_of_contents)

##<a name='modified_module_pattern'></a>modified module pattern:
```js
  var foo = (function() {
    var publicAPI = {
      bar: function() {
        publicAPI.baz();
      },
      baz: function() {
        console.log('baz');
      }
    };
    return publicAPI;
  })();
  foo.bar();   // `baz`
```

[Back to table of contents](#table_of_contents)

## <a name='modern_module_pattern'></a> Modern module pattern:
```js
  define('foo', function() {
    var o = {bar: 'bar'};

    return {
      bar: function() {
        console.log(o.bar);
      }
    };
  });
```
we have a named module `foo` and we are doing stuff on
that.

This pattern when used using `IIFE` pattern, can only 
provide one instance, meaning its only for `one time use`.

  On the other hand if we use this pattern by using common
  function and then assign the return value to different 
  variable, we will have different instance of the same module.

Ex:
```js
  var foo = function() {
    var publicAPI = {
      bar: function() {
        publicAPI.baz();
      },
      baz: function() {
        console.log('baz');
      }
    };
    return publicAPI;
  };
  var myFoo = foo();
  var yourFoo = foo();
```

[Back to table of contents](#table_of_contents)

##<a name='object_orientation'></a>Object Orienting:  
1. <a name='prototype'></a>Prototype:
    ```js
        function Foo(who) {
          this.me = who;
        }
        Foo.prototype.identify = function() {
          return 'I am ' + this.me;
        };
        
        var a1 = new Foo('a1');
        var a2 = new Foo('a2');
        
        a2.speak = function() {
          alert('Hello, ' + this.identify() + '.');
        };
        a1.constructor === Foo;
        a1.constructor === a2.constructor;
        a1.__proto__ === Foo.prototype;
        a1.__proto__ === a2.__proto__;
    ```
    Every single `object` is built by a constructor 
    function.
    Each time a constructor is called, a new object is
    created.
    A constructor makes an object `based on` its own
    prototype.`based on` is not completely true in case of 
    JavaScript.
    `based on` implies that we take the prototype and 
      we stamp out a copy of it.
  
    This is true in `class` oriented languages.
    so `based on` give us the wrong idea.
    
    A more appropriate way of saying the above thing is:
    >A constructor makes an object `linked to` its own
      prototype.

    **Note**:
    >While discussing the `new` keyword we said in 
      the `second` step that it linked to an object.
    
    **Note**:
    >Before anything gets executed in the above code
    we already have somethings, which are:
    >1. A function called `Object` (capital `O`)
    >2. An object which does not have any name, but a
      label named `Object.prototype`.

    >Object func ----`.prototype`------> unnamed object
    
    >The `Object` function has been linked to the object
    which does not have any name.  
    On the unnamed object, we have functions like
    `toString` and several values which are built in
    the language.

    When the first line of above code gets executed:
    1. We will have a function called `Foo`,         
    2. Its also going to create an `object` that 
       we are linked to, and it will have the same 
       arbitrary name: `.prototype`.
       >Foo func -------`.prototype`-------> unnamed object
          
    Also the unnamed object is linked to the 
    unnamed object of the 'Object' function and this
    linkage is labeled as `[[p]]`.
    (`[[p]]` is explained later in this notes)

    3. In addition to the above connection, there is 
       also a connection in the opposite direction.
    The unnamed object has a property on the `function`
    called `.constructor`.

    Foo function <------`.constructor`---- unnamed object

    Most people think the '.constructor' means is
    constructed by. In other words the unnamed object 
    is constructed by the `function`.`But its not true`.

    The word `constructor` is an arbitrary word
    it could have been any other random word.

    So there is a `two-way` linkage. Now when we execute the
   `Foo.prototype.identify = function()` line, we put the
   `identify` property directly on the `unnamed` object.

    Now coming on the code `var a1 = new Foo('a1')`
    Here we have encountered the `new` keyword.
    So four things will happen:
    1. brand new object will gets created.
    2. object gets linked to another object.
      (so the newly created object will get linked
        to the `unnamed object.`)
    3. The context gets set to the `this`. So the newly
      created object will have a property called `me`,
      which will have the value `a1`.
    4. We return `this`, which gets assigned to the 
      variable `a1` in the code.
      So now the name of the newly created object will 
      be `a1`. 

      Now executing `a2.speak = function()`
      This will put the `speak` property on the object
      which is `a2`.

      So at this point of time, if we try to execute 
      `a1.speak()`, it wont get executed as there is
      no property called `speak` on the `a1` object.

      Coming on to the code `a1.constructor === Foo`.
      There is no direct property called `constructor`
      on the `a` object.
      Some people think that there is a hidden property
      called `constructor` on the `a1` object, but in
      reality its not there.
      
      So when we execute the above code, as there is 
      no property called `constructor` on the `a1` object
      it will go up in the `Prototypical` chain.
      And the unnamed object has a linkage of `.constructor`
      so the call will end of being `Foo`.
      So `a1.constructor is Foo`.

      The linkage of newly created object, such as `a1`,
      with the unnamed object, which is linked with the
      function such as `Foo` using `.prototype` label, is
      called `Prototypical linkage`.
      In the specs this linkage is denoted by `[[Prototype]]`
      But in this notes we will say it `[[P]]` instead.

      They are internal linkage, they are not visible to 
      public.

      Now when we execute `a1.__proto__ === Foo.prototype`
      The name for `__proto__` is `dunder`.
      The pronunciation of the above property is
      `dunder proto property`.
      So when the above code will get executed it will check
      whether `a1` have a `dunder proto` property.
      
      As it does not have, it will go up in the Prototypical
      chain and check the same thing on the unnamed object.
      As that object also does not have the `dunder proto`
      property, so we will go up in the Prototypical chain,
      which happens to be the unnamed object of the 
      `Object` function.
      
      Now on this object there is a property named
      `__proto__`.
      It turns out that it is not a property, rather its
      a `getter` function. So the above code is a 
      `function call`.
        
      This function returns the `internal prototype` linkage
      of whatever the `this` binding is.

      So when we have called the `__proto__` function, the 
      `this` keyword is `a1`.

      So this function `returns the internal prototype` linkage
      of `a1`.

      So the `[[P]]` was the internal linkage and `__proto__`
      is the public linkage of `a1` with the unnamed object
      of the `Foo` function.

      So `__proto__` is the public property that references
      the internal characteristics.

      The problem with `dunder proto` is, it never been
      standardised.

      But its a de-facto standard for every other browser
      except the `IE`.

      So we can see that `a1.__proto__` is same as 
      `Foo.prototype`.
      The same goes for `a2`.

      As of `ES5`, there is an standard utility
      ```js 
        a1.__proto__ === Object.getPrototypeOf(a1);
      ```
  
      So this utility extract the internal `Prototypical
      linkage` for us.

      So this utility is there for `IE > 8`
      But what we will do for browsers < 8.  
        
      Sol:
      ```js
        a2.__proto__ == a2.constructor.prototype;
        // for IE < 8

      ```

      But the problem is, `both the property`
      `the constructor property` and `the prototype`
      property are `writable` property. They happen 
      to default to pointing where we discussed above.
      But when they gets over written, the above code
      for `IE < 8` is unreliable.
       
      ```js
        function foo(who) {
          this.me = who;
        }
        Foo.prototype.identify = function() {
          return 'I am ' + this.me;
        };

        function Bar(who) {
          Foo.call(this, who);
        }
        // Bar.prototype = new Foo(); // or...
        Bar.prototype = Object.create(Foo.prototype);
        // NOTE: .constructor is broken here, need to fix
        
        Bar.prototype.speak = function() {
          alert('Hello, ' + this.identify() + '.');
        };

        var b1 = new Bar('b1');
        var b2 = new Bar('b2');

        b1.speak(); // alerts: `Hello, I am b1.`
        b2.speak(); // alerts: `Hello, I am b2.`
      ```

      In the above code we wanted the class `Bar` to
      be the child of the `Foo` class. As we have written 
      the function names in CAPS, they refer to the class.

      We can achieve this by assigning what is there in 
      the comment in the code `// Bar.prototype = new Foo();`
      But by this way we will un-wantedly `call Foo every` time.

      >Soln is use the `Object.create`.
      The `Object.create` does the first two things of what
      `new` keyword does.

      In this code, when we call `var b1 = new Bar('b1')`.
      `this` will point to `b1`
      So when we call `b1.speak();` it will check whether
      `b1` is having a `speak` property. No it does not have.
      
      Then it will go Prototypically up, and check whether
      `Bar.prototype` have `speak` property, and it is 
      there on the `Bar`, so it will call it.

      Now in the speak property, we have `this.identify`,
      here `this` points to `b1`, so we check whether `b1`
      has a `identify` property, as it does not have, we 
      will go Prototypically up, and check if 
      `Bar.prototype` has a identify property.
      
      No, so we traverse further up, and check whether
      `Foo.prototype` has a 'identify' property, as it has
      it gets called.

      But there is a drawback in this approach.
      We have lost the `.constructor` linkage. But how?
      As of line 6 in the above code we have the 
      `.constructor` property but when we 
      execute the code on line 12, we change the 
      reference of the prototype object, and the 
      newly created one does not have a `.constructor`
      property.
      
      So we will delegate up to check `does the foo`
      prototype has a `.constructor`, yes it does and it
      is pointing to the `Foo`, so we get `Foo` instead
      of `Bar`.

      So this bizzare result proves that `.constructor`
      does not mean `is constructed by`, it is simply 
      an arbitrary property.

      Now we can use the class pattern rather than the `module pattern`.
      For this all the methods which were private in the 
      module pattern will now come to the `prototype of the class`.

      It is necessary to write the prototype like:
      `ClassName.prototype.someMethod`
      Because when we write like:
      ```js
          ClassName.prototype = {
            someMethod: 'hello'
          }
      ```
      
      We are actually overriding the prototype and thus we will
      loose the `.constructor` property.

      Generally we should always use the dynamic binding
      provided to us using 'this' but there is one instance
      where we have to fallback to the lexical way of 
      doing things.  
      
      consider the below example:
      ```js
        NotesManager.prototype.showHelp = function() {
          this.$help.show(); // $help is just a convention 
                            // for saying that this variable
                            // belongs to JQuery.
          
          document.addEventListener('click', function __handler__(evt){
            evt.preventDefault();
            evt.stopPropagation();
            evt.stopImmediatePropagation();

            document.removeEventListener('click', __handler__, true);
            this.hideHelp();
          //}, true) 
            }.bind(this),true)                
        };
      ```

      Now when we call this function, the `this` in the 
      last line of code `will not point` to the reference
      we want, rather it will be the `button` on which
      the listener is being called.

      To solve this problem we can use `.bind` function.
      So we do like the uncommented last line.

      But in the event listener function we are 
      `removing the event listener` using the `named` method.
      But due the `hard binding` the `named function` is `not` 
      there.

      So only in this case we do like below:
      ```js
          NotesManager.prototype.showHelp = function() {
            var self = this;
            self.$help.show(); // $help is just a convention 
                              // for saying that this variable
                              // belongs to JQuery.
            
            document.addEventListener(`click`, function __handler__(evt){
              evt.preventDefault();
              evt.stopPropagation();
              evt.stopImmediatePropagation();
    
              document.removeEventListener(`click`, __handler__, true);
              self.hideHelp();
            }, true); 
      ```

      But in general this is not a good practice.

[Back to table of contents](#table_of_contents)

2. <a name='inheritance'></a> Inheritance:
    >Inheritance means that the child is having the copy of 
    what the parent have.

    So its not wise to call `prototypal inheritance` for 
    JavaScript as we are not using the word inheritance for
    its `correct meaning`.

    In JavaScript when we have the object arbitrarily 
    called `Foo.prototype` and when we have `a1` and `a2`
    object, they are linked in the opposite way to what
    inheritance does.
    In inheritance we have the below structure:  
    
      >`parent -----------> child`
      
      meaning child is a copy of the parent.
    But in JS:
      >`Foo.prototype <----------- child`
      
      meaning the child is behaviorally linked
      to the prototype.

    This delegation in JS is a `design pattern` and it is
    called `Behavior Delegation`.

[Back to table of contents](#table_of_contents)

3. <a name='oloo'></a> OLOO: (Objects Linked to Other Objects)
    ```js
    
    function Foo(who) {
      this.me = who;
    }

    Foo.prototype.identify = function() {
      return 'I am ' + this.me;
    };

    function Bar(who) {
      Foo.call(this, who);
    }

    Bar.prototype = Object.create(Foo.prototype);

    Bar.prototype.speak = function() {
      alert('Hello, ' + this.identify() + '.');
    };

    var b1 = new Bar('b1');
    b1.speak();
    ```

    In this code we only need to care about the three 
    objects which are prototypically linked.

    `b1` is linked to `Bar.prototype`.
    `Bar.prototype` is linked to `Foo.prototype`.

    Now the same thing can be achieved by using only the
    `objects` rather than the `constructor`.

    As a first step towards refinement we will remove 
    the `new` keyword.
    The refined code will look like the below:
    ```js
      function Foo(who) {
        this.me = who;
      }

      Foo.prototype.identify = function() {
        return 'I am ' + this.me;
      };

      function Bar(who) {
        Foo.call(this, who);
      }

      Bar.prototype = Object.create(Foo.prototype);

      Bar.prototype.speak = function() {
        alert('Hello, ' + this.identify() + '.');
      };

      var b1 = Object.create(Bar.prototype);
      Bar.call(b1, 'b1');
      b1.speak();
    ```
        [js]

    In this refinement remove the `new` keyword and
    instead use the `Object.create` utility to 
    create a `brand new object` and `link` the object to 
    the `prototype`.

    Then we explicitly bind the `Bar` `this` to `b1`.

    Consider the second refinement below:
    ```js
      function Foo(who) {
        this.me = who;
      }

      Foo.prototype.identify = function() {
        return 'I am ' + this.me;
      };

      //changed
      var Bar = Object.create(Foo.prototype);
      Bar.init = function(who) {
        Foo.call(this, who);
      };

      // changed
      Bar.speak = function() {
        alert('Hello, ' + this.identify() + '.');
      };

      var b1 = Object.create(Bar); // changed
      b1.init('b1'); // changed
      b1.speak();
    ```

    In this refinement, we have ditched the function `Bar`
    and made `Bar` an object instead.

    Consider the third refinement:
    ```js
     var Foo = {
       init: function(who) {
         this.me = who;
       },
       identify: function() {
         return 'I am ' + this.me;
       }
     };

     var Bar = Object.create(Foo);    
     Bar.speak = function() {
       alert('Hello, ' + this.identify() + '.');
     };

     var b1 = Object.create(Bar); 
     b1.init('b1'); 
     b1.speak();
    ```
    Now these are peer objects that can delegate to 
    each other.

[Back to table of contents](#table_of_contents)

3. <a name='object_create'></a> Object.create:
    ```js
      if (!Object.create) {
        Object.create = function(o) {
          function F() {}
          F.prototype = o;
          return new F();
        };
      }      
    ```
[Back to table of contents](#table_of_contents)

## <a name='async_patterns'></a> Async Patterns:
* Callbacks
* Generators / Co-routines
* Promises

<a name='callbacks'></a>**Callbacks**:  
   * `callback hell`:
        ```js
          setTimeout(function(){
            console.log('one');
            setTimeout(function(){
              console.log('two');
              setTimeout(function(){
                console.log('three');
              }, 1000);
            }, 1000);
          }, 1000);
        ```

      **Note**:
      >callback hell does not have anything to do with 
        nesting.

      Consider the below code:
        ```js
          function one(cb) {
            console.log('one');
            setTimeout(cb, 1000);
          }

          function two(cb) {
            console.log('two');
            setTimeout(cb, 1000);
          }

          function three() {
            console.log('three');
          }

          one(function(){
            two(three)
          });
        ```

   * `Inversion of control`:
        >When we loose control over a program to let it
        execute by some third party library.

   * `Solving callback problems`:
      1. separate callbacks:
            ```js
              function trySomething(ok, err) {
                setTimeout(function(){
                  var num = Math.random();
                  if (num > 0.5) ok(num);
                  else err(num);
                }, 1000);
              }
    
              trySomething(
                function(num){
                  console.log('Success: ' + num);
                },
    
                function(num){
                  console.log('Sorry: ' + num);
                }
              );
            ```
            In this we expect the third party library to
            call one method when there is  `success` and 
            the other when there is `failure`.
            But this is more `implicit trust`, because
            we are trusting on them that they will call
            `only one method`. But what will happen when they
            call both the methods: `Our code will break`.

      2. error-first style:
            ```js
              function trySomething(cb) {
                setTimeout(function(){
                  var num = Math.random();
                  if (num > 0.5) cb(null, num);
                  else cb('Too low!');
                }, 1000);
              }
    
              trySomething(function(err, num){
                if (err) {
                  console.log(err);
                }
                else{
                  console.log('Number: ' + num);
                }
              });
            ```
      
            In this code we have only  one function.
            So we are only checking for the `error` object.
    
            But consider a scenario when they return an `error`
            object and then a `success` value.
            In this situation we will `reject` the `success` value
            as we are only checking for the `error` object.

  
<a name='generators'></a>**Generators (yield) (`ES6`)**:
   ```js
        function* gen() {
          console.log('hello');
          yield null;
          console.log('World');
        }
    
        var it = gen();
        it.next(); // prints `Hello`
        it.next(); // prints `World`
   ```
   So calling the `gen` function, creates an `iterator`.
   so when we call `it.next()` it will start from the 
   line 2 and execute till it encounters a `yield` statement.
   The second `it.next()` will also run till it encounters
   a `yield` statement or till end of the program.

   `yield` is used for `two way message passing`.
   meaning we can pass value to `yield` from outside
   and `yield` can also return value to the calling function.

   Consider the below code:
   ```js
    var run = coroutine(function* (){
      var x = 1 + (yield null);
      var y = 1 + (yield null);
      yield(x + y);
    });

    run();
    run(10);
    console.log('something: ' + run(30).value);
   ```

  So when first `run()` will be called, the `yield` will
  return `null` to the function.

  When we call `run(10)`, the value of the previous `yield` 
  expression will be 10.
  So the value of `x = 11`.

  and it encounters the next `yield` and `null` will get
  returned to `run(10)`

  now when `run(30)` will get executed, 30 will get
  substituted for the `yield` expression and 
  `y = 31`.

  Then it encounters the next `yield` and (x + y) will
  be returned which is `42`.

  Till now everything was looking `synchronous` in 
  generators.

  Consider the below code:
   ```js
    function getData(d) {
      setTimeout(function() {run(d);}, 1000);
    }

    var run = coroutine(function* (){
      var x = 1 + (yield getData(10));
      var y = 1 + (yield getData(30));
      var answer = (yield getData('something: ' + (x + y)));
      console.log(answer);
    });

    run();
   ````

<a name='promises'></a>**Promises**:
   >JQuery style Promises:
   ```js
        var wait = jQuery.Deferred();
        var p = wait.promise();

        // this happens when we listen for an event
        // continuation event
        p.done(function(value) {
          console.log(value);
        });

        // when we call resolve on Deferred, it will
        // automatically fires the 'done' event for 
        // any promises that are listening to it.
        setTimeout(function(){
          wait.resolve(Math.random());
        }, 1000);
   ```
   
   Ex:
   ```js
        function waitForN(n) {
          var d = $.Deferred();
          setTimeout(d.resolve, n);
          return d.promise();
        }
    
        // promise gets returned from the 
        // waitForN function, and when we call
        // '.then' we are listening for the 
        // continuation event on that returned promise.
        waitForN(1000).then(function() {
          console.log('hello world');
          return waitForN(2000);
        })
        .then(function(){
          console.log('finally!');
        });

   ```

   Promises un-invert the `inversion of control`
  

   **Prob**:
    Make three request to fetch some data from files
    asynchronously, but display them in the given order.

   **Sol**:
    For this problem we need to track the internal 
    state of the call, if we are using `callbacks`.

[Back to table of contents](#table_of_contents)

**How event loop works in JS**:
   >Our browser have following things:
   >1. call stack (js runtime)
   >2. WebAPIs (ex: setTimeout, DOM(document), ajax(XMLHttpRequest))
   >3. event loop

  Actually JS is `single threaded`.
  So when we execute some things such as the below code:
  ```js
      console.log('hiii there');
      function foo() {
        console.log('hello');
      }
      function bar() {
        console.log('there');
        foo();
      }
      function baz() {
        console.log('hi');
        bar();
      }
      baz();
  ```

  How these things are executed by the JS runtime is:
   >1. At first `main()` will appear in the stack.
        `main()` is the anonymous function corresponding to the
        file itself.
   >2. then `console.log('hiii there')` will appear in the
        `stack`.
   >3. then `baz()`
   >4. then `console.log('hi')`
   >5. then `bar()`
   >6. then `console.log('there')`
      and like that.

  But consider the below code:
  ```js
      console.log('hii there');
      setTimeout(function foo() {
        console.log('hello there')
      });
      console.log('hii')
  ```

  How will this code gets executed?
  >When we check the 'call stack' we will find:
  >1. console.log(`hii there`) will get executed.
  >2. setTimeout(function foo() {
        console.log('hello there')
      }); will get executed but it will disappear from the 
      stack before executing console.log('hello there').
  >3. then console.log(`hii`) will get executed.

  But where does that setTimeout function went?
  >Actually `setTimeout` is a function defined in `WebAPI`
  not in actual `JS runtime`.  
  >So the function after leaving the `call stack` will 
  enter into the `WebAPI` and that `api` will keep track
  of the `timer`.
  When the timer is complete, as it can not directly modify
  the `call stack`, it will throw the function into the 
  `event loop` or `task queue`.
  `Event loop` has to wait till the stack is clear, before 
  it can push the `callback` on the `call stack`.
  `setTimeout` is not a guaranteed time of execution, it is 
  `minimum time` of execution.

  When we block the stack with some `slow` work, the 
  browser does not render, (render queue gets stopped).

  So during that time, nothing will work, no buttons, no
  selection, nothing.

  Where in case of `asynchronous call`, browser gets a chance
  to re-render after every call.
