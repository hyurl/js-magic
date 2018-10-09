# JS-Magic

**Lets classes support magic methods in JavaScript based on ES6 `Proxy`.**

We know that ES6 brings the capablity of Proxy that allows us observing an 
object, and setters ang getters are build-in support in JavaScript, but if we 
need to build those things every time, that's just mass and pain. With this 
package, you can define setters and getters, along with other funcitons, right 
in the class difinition itself, and when you instantiate the class, the instance
will always have the the benifits of the magical calling functionalities.

This package is inspired by PHP magic methods, it currectly support sthese migic 
methods: `__get`, `__set`, `__has`, `__delete`, `__invoke`. Other methods like 
`toString` and `toJSON` are built-in support in JavaScript.

## Install

```sh
npm i js-magic
```

## Example

```typescript
// this example is coded in TypeScript, be aware of the difference between TS 
// and JS. All the magic methods are optional, but here I'll show all the usage 
// of them.

import { applyMagic, MagicalClass } from "js-magic";

@applyMagic
export class Car implements MagicalClass {
    name!: string;
    wheels?: number;

    constructor(name?: string, wheels?: number) {
        if (name !== undefined) this.name = name;
        if (wheels !== undefined) this.wheels = wheels;
    }

    /**
     * If a property doesn't exist, returns `null` instead of `undefined`, and 
     * if the property is 'name', returns it according to the class name plus 
     * 'Instance'. 
     */
    protected __get(prop: string | symbol): any {
        return prop in this ? this[prop]
            : (prop == "name" ? this.constructor.name + " Instance" : null);
    }

    /** If the property is name, appends it with 'Instance'. */
    protected __set(prop: string | symbol, value: any): void {
        this[prop] = prop == "name" ? value + " Instance" : value;
    }

    /**
     * Ignores the properties starts with '__', and always returns `true` when 
     * testing 'name'.
     */
    protected __has(prop: string | symbol): boolean {
        return (typeof prop != "string" || prop.slice(0, 2) != "__")
            && (prop in this || prop == "name");
    }

    /** If the property starts with '__' or is 'name', DO NOT delete. */
    protected __delete(prop: string | symbol): void {
        if (prop.slice(0, 2) == "__" || prop == "name") return;
        delete this[prop];
    }

    /**
     * This method will be called when the class is invoked as a function. You 
     * may be a little confused since being told that ES6 class cannot be called
     * as function, AKA without `new` operator, well, when using this package, 
     * you CAN. But remember, it is called as a pure function, means the `this` 
     * variable is not set.
     */
    protected __invoke(...args: any[]): any {
        return "invoking Car as a function";
    }
}
```

## How It Works?

The decorator `applyMagic` is a function that returns a highly-comstomized ES5 
psudo-class, it will replace the original class, so that when instantiating, the 
magic methods will be auto-applied to the instance wrapped by a `Proxy`. Since 
`applyMagic` is a function, so if you're coding in JavaScript without decorator
support, you can manually call it to generate the wrapping class and assign to 
the old one. Like this:

```javascript
const { applyMagic } = require("js-magic");

class Car {
    // ...
}

Car = applyMagic(Car);
```

Since the returns class is wrapped in ES5 style, so that it allows you calling 
as a function, which the `__invoke` method will take place.

## Support of Inheritance

This package also supports native inheritance, allows you inheriting the magical
calling functionalities from a super class to sub-classes. BUT, unless you also
decorate with (or manually call) `applyMagic` on the sub-class, otherwise 
the `__invoke` method will never be performed on that sub-class.

Also you can rewrite the magic methods in the sub-class, and call the super's 
via `super` keyword.

## Additional Symbols

This package also provides symbols according to the magic method names (`__get`, 
`__set`, `__has`, `__delete`, `__invoke`), you can use them if you want to hide 
the methods from IDE IntelliSence, but gernally they are not common used. And 
you've already seen that in the example all the magic methods are defined as 
`protected`, so they will not be seen outside the class and its siblings.

## Supported Environments

Any environment that supports ES6 `Proxy` will run this package perfectly, 
generally, NodeJS `6.0+` and modern browsers (`IE` aside) should support `Proxy`
already.

In browsers, if you're not using any module resolution, access the global 
variable `window.magic` instead.

## More Examples

For more examples, please check out the [Test](./test.js).