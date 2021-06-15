# Context
_whose grandma is it anyway?_

> Hopefully an example will make it easier to see...

```ts
class A {
    private myName: string = "I am A";
    public method: Function;
}

class B {
    private myName: string = "I am B";

    public printName() {
        // Here... what does "this" means?!
        console.log(this.myName);
    }
}

const a = new A();
const b = new B();

// I assign a.method
a.method = b.printName;
a.method(); // This will print "I am A"
b.printName(); // This will print "I am B"

// This will create a new function where `b` is the `this` for that function
a.method = b.printName.bind(b);
a.method(); // This now prints "I am B"
```

> They are calling the same method... and getting different results!?

Imagine you have a phone number that calls your grandma _Esther_. Let's say that this number is `555-1234`. When **you** dial `555-1234` you call **your** grandma.  
Now you dictate me the number, `555-1234` and **I** dial, it calls **my** grandma _Rachel_... Wait what?!  
We saw a moment ago that to reference a method or member of a class we need to prepend the `this.` keyword. In javascript, the `this` object, will become "whoever owns the function" (the context of who and where called the function).  
So when you need to pass a method as a parameter (let's call it `b.printName`), when that function parameter gets called, inside the code of that function, `this` is not `b`!
To fix this, you need to tell the function to `bind` (attach) the `this` (context) object. This is done by adding `.bind(b)` to the function like so: `b.printName.bind(b)`  
Back to the telephone example, for **me** to call **your** grandma _Esther_ you give me `555-1234.bind(estherGrandson)` and that makes it so that when I use the number, it understands that the original owner it's you and thus calls your grandma _Esther_.

This might make no sense now but you will start seeing a lot of `.bind()` in future chapters and this is the primer.

If you want a more formal explanation with fewer grandmas, you can try [MDN webdocs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)

<aside class="notice">
PixiJS events also have another way to bind the context. It will be most likely a parameter of the method that requests a function. Setting either will work, no need to set it both ways.
</aside>