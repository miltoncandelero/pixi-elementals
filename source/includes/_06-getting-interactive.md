# Getting interactive

In web HTML5 games we usually rely on 2 kinds of inputs: Some sort of pointing device (be it a mouse or a touchscreen) and the keyboard.  

We will explore how to use both of them and then we will do a quick overview on how you can trigger some of your own custom events.

--
## Pointer Events
_mouse... touch.... both?_

```ts
import { Container, FederatedPointerEvent, Sprite } from "pixi.js";

export class Scene extends Container {
    private clampy: Sprite;
    constructor(screenWidth: number, screenHeight: number) {
        super();

        this.clampy = Sprite.from("clampy.png");

        this.clampy.anchor.set(0.5);
        this.clampy.x = screenWidth / 2;
        this.clampy.y = screenHeight / 2;
        this.addChild(this.clampy);

        // events that begin with "pointer" are touch + mouse
        this.clampy.on("pointertap", this.onClicky, this);

        // This only works with a mouse
        // this.clampy.on("click", this.onClicky, this);

        // This only work with touch
        // this.clampy.on("tap", this.onClicky, this);

        // Super important or the object will never receive mouse events!
        this.clampy.interactive = true;
    }

    private onClicky(e: FederatedPointerEvent): void {
        console.log("You interacted with Clampy!")
        console.log("The data of your interaction is super interesting", e)

        // Global position of the interaction
        // e.global

        // Local (inside clampy) position of the interaction
        // clampy.toLocal(e.global)
        // or the "unsafe" way:
        // (e.target as DisplayObject).toLocal(e.global)
        // Remember Clampy has the 0,0 in its center because we set the anchor to 0.5!
    }
}
```

PixiJS has a _thing_ that whenever you click or move your mouse on the screen it checks what object were you on, no matter how deep in the display list that object is, and lets it know that _a mouse_ happened.  
(If curious, that _thing_ is a plugin called [Federated Events System](https://pixijs.download/dev/docs/PIXI.EventSystem.html))  

The basic anatomy of adding an event listener to an imput is:  
`yourObject.on("stringOfWhatYouWantToKnow", functionToBeCalled, contextForTheFunction)`

and the second super important thing is:  
`yourObject.interactive = true`

### Touch? Mouse? I want it all!  
The web has moved forward since its first inception and now we have mouses and touchscreens!  
Here is a small list of the most useful events with their mouse, touch, and catch-all variants.  
The rule of thumb is that if it has `pointer` in the name, it will catch both mouse and touch.  

| Mouse only  | Touch only   | Mouse + Touch |
|-------------|--------------|---------------|
| `click`     | `tap`        | `pointertap`  |
| `mousedown` | `touchstart` | `pointerdown` |
| `mouseup`   | `touchend`   | `pointerup`   |
| `mousemove` | `touchmove`  | `pointermove` |

<aside class="warning">Still not working? You forgot the <code>.interactive = true</code>, didn't you?</aside>

### The event that fired

When your function gets called you will also receive a parameter, that is all the data that event produced. You can see the [full shape of the object here](https://pixijs.download/dev/docs/PIXI.FederatedPointerEvent.html).  
I will list now some of the most common properties here now:  

* `event.global` This is the global (stage) position of the interaction
* `(e.target as DisplayObject).toLocal(e.global)` This is the local (object that has the event) position of the interaction. It has an unsafe cast due to the new Event System being a bit paranoid.
* `event.pointerType` This will say `mouse` or `touch`

### Old Magical stuff (pre PixiJS v7)

You might have come across by the fact that if you have a function that is called exactly the same as an event (for example a `click` function) and your object is interactive, that function gets called automagically.  
That was because there is a line inside the old [Interaction Manager's](https://pixijs.download/v6.5.8/docs/packages_interaction_src_InteractionManager.ts.html) that if it finds that a display object has a method with the same name as an event it calls it.

**This was removed on PixiJS' v7 new [Federated Events System](https://pixijs.download/dev/docs/PIXI.EventSystem.html)**

--

## Keyboard

```ts
import { Container, Sprite } from "pixi.js";

export class Scene extends Container {
    private clampy: Sprite;
    constructor(screenWidth: number, screenHeight: number) {
        super();

        // Clampy has nothing to do here, but I couldn't left it outside, poor thing
        this.clampy = Sprite.from("clampy.png");
        this.clampy.anchor.set(0.5);
        this.clampy.x = screenWidth / 2;
        this.clampy.y = screenHeight / 2;
        this.addChild(this.clampy);

        // No pixi here, All HTML DOM baby!
        document.addEventListener("keydown", this.onKeyDown.bind(this));
        document.addEventListener("keyup", this.onKeyUp.bind(this));
    }

    private onKeyDown(e: KeyboardEvent): void {
        console.log("KeyDown event fired!", e);

        // Most likely, you will switch on this:
        // e.code // if you care about the physical location of the key
        // e.key // if you care about the character that the key represents
    }

    private onKeyUp(e: KeyboardEvent): void {
        console.log("KeyUp event fired!", e);

        // Most likely, you will switch on this:
        // e.code // if you care about the physical location of the key
        // e.key // if you care about the character that the key represents
    }
}
```

And here is where PixiJS lets go of our hands and we have to grow up and use the DOM.

Luckily, the DOM was meant to use keyboards and we have two big events: `keydown` and `keyup`. The bad news is that this detects **ALL** keypresses, **ALL** the time. 

To solve _which_ key was pressed at any point in time, we have two string properties on the keyboard event: *code* and *key*.  

### key
`key` is the easiest one to explain because it is a string that shows what character should be printed into a textbox after the user presses a key. It follows the user keyboard layout and language so if you need the user to write some text, this is the ideal property.

### code
`code` might be confusing at first, but let me tell you bluntly: Not everybody uses QWERTY keyboards. AZERTY and DVORAK keyboards are a thing (and we are not even getting into right to left keyboards) so if you bind your character jumping to the `Z` _key_ you might find out later that not everybody has it in the same place in their keyboard distributions!  
For problems like this, `code` was born. It represents a physical location in a keyboard so you can confidently ask for the `Z` and `X` keyboard keys and they will be one next to the other no matter what language it is using.

<aside class="info">To see the <code>key</code> and <code>code</code> for any key on your keyboard, check <a href="https://keycode.info/">this awesome website</a></aside>

> Consider this a `Keyboard.ts` recipe:

```ts
class Keyboard {
    public static readonly state: Map<string, boolean>;
    public static initialize() {
        // The `.bind(this)` here isn't necesary as these functions won't use `this`!
        document.addEventListener("keydown", Keyboard.keyDown);
        document.addEventListener("keyup", Keyboard.keyUp);
    }
    private static keyDown(e: KeyboardEvent): void {
        Keyboard.state.set(e.code, true)
    }
    private static keyUp(e: KeyboardEvent): void {
        Keyboard.state.set(e.code, false)
    }
}
```

But sometimes we need to know the _state_ of a key: Is it pressed? To do this we need to keep track of this state of every key manually, luckily we can do it with a really simple static class.

We just need to once call the `Keyboard.initialize()` method to add the events and then we can ask `Keyboard.state.get("ArrowRight")` and if this says true then the key is down!

<aside class="warning"> If you are an old school programmer you might recognize a <code>Map</code> object and try to access it with brackets notation: <code>Keyboard.state["ArrowRight"]</code>. However <b>this won't work with javascript maps!</b></aside>

### keyCode, which, keypress are deprecated!
<aside class="warning"> If you are an old web programmer you might be tempted to fall back to your old <code>keyCode</code> or <code>which</code> and the <code>keypress</code> event, but those have been deprecated and you <b>shouldn't be using them.</b></aside>

--

## Making custom events and the overall syntax

> All display objects are event emitters so we can create a sprite to test:

```ts
const clampy: Sprite = Sprite.from("clampy.png");
clampy.on("clamp", onClampyClamp, this);

clampy.once("clamp", onClampyClampOnce, this);

// clampy.off("clamp", onClampyClamp); // This will remove the event!


// somewhere, when the time is right... Fire the clamp!
clampy.emit("clamp");


// If you come from c++ this will mess you up: Functions can be declared after you used them.
function onClampyClamp() {
	console.log("clampy did clamp!");
}

function onClampyClampOnce() {
  console.log("this will only be called once and then removed!");
}
```

> using `.off(...)` can be tricky. If you used `.bind(this)` then it will probably not work. That is why there is an extra parameter so you can provide the `this` context to the `on(...)` function!

What do we do if we want to be notified when something happens? One way could be setting a boolean flag and looking at it at every update loop, waiting for it to change but this is clumsy and hard to read. Introducing Events!

An event is a way for an object to _emit_ a scream into the air and for some other object to _listen_ and react to this scream.  
For this purpose, PixiJS uses EventEmitter3.  
[The API is made to mirror the one found on node.js](https://nodejs.org/api/events.html)  
Let's see a quick overview of how EventEmitter3 works:  

* `.on(...)`: Adds an event listener.
* `.once(...)`: Adds an event listener that will remove itself after it gets called once.
* `.off(...)`: Removes an event listener. (Tricky to use if you use `.bind`!)
* `.emit(...)`: Emits an event, all listeners for that event will get called.
* `.removeAllListeners()`: Removes all event listeners.

<aside class="warning">
An event listener will keep an object alive. Calling <code>.destroy()</code> will remove all listeners. Otherwise you need to call <code>.off(...)</code> for each event (or call <code>.removeAllListeners()</code>)
</aside>