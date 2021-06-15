# Animating stuff
_shake it baby!_

Let's sum it up quickly, we have 3 main ways of animating stuff:

* **Animated Sprites**: Frame-by-frame animations made out of multiple textures.
* **Ticker and some Kinematic equations**: During each frame, we check the speed of something and move it accordingly.
* **Tweens**: Short for "in-betweens", it's a way of telling an object "go to that place in this many seconds" and have it go without having to think the physics.

There is another kind of animations called bone animation and while there are PixiJS plugins for loading two of the biggest formats out there, Spine and DragonBones, we won't be seeing them here.

## AnimatedSprite

> Don't freak out, we will use some javascripty stuff.

```ts
import { AnimatedSprite, Container, Texture } from "pixi.js";

export class Scene extends Container {
    constructor() {
        super();

        // This is an array of strings, we need an array of Texture
        const clampyFrames: Array<String> = [
          "clampy_sequence_01.png",
          "clampy_sequence_02.png",
          "clampy_sequence_03.png",
          "clampy_sequence_04.png"
        ];

        // `array.map()` creates an array from another array by doing something to each element.
        // `(stringy) => Texture.from(stringy)` means
        // "A function that takes a string and returns a Texture.from(that String)"
        const animatedClampy: AnimatedSprite = new AnimatedSprite(clampyFrames.map((stringy) => Texture.from(stringy)));
        // (if this javascript is too much, you can do a simple for loop and create a new array with Texture.from())

        this.addChild(animatedClampy); // we just add it to the scene

        // Now... what did we learn about assigning functions...
        animatedClampy.onFrameChange = this.onClampyFrameChange.bind(this);
    }

    private onClampyFrameChange(currentFrame): void {
        console.log("Clampy's current frame is", currentFrame);
    }
}
```

> Clampy sequence assets don't exist. I lied to you. You will need your own assets.

Frame-by-frame animations go with 2d games like toe and dirt. Animated sprite has got your back. (This inherits from `Sprite`)  
[PixiJS AnimatedSprite API](https://pixijs.download/dev/docs/PIXI.AnimatedSprite.html)  
Tips and stuff:

* You make this by passing an array of `Texture` objects.
  * A good way to do this is by loading a spritesheet directly. We will touch that on the [loader recipe](#recipe-preloading-assets).
* The constructor has a parameter `autoUpdate` that comes by default on true.
  * If you set it to false, you need to use `.update(...)` manually.
* A bunch of things that do exactly what they say...
  * `.loop`
  * `.currentFrame`
  * `.playing`
  * `.play()`, `.stop()`, `.gotoAndPlay()` and `.gotoAndStop()`
* `.onFrameChange`: Assign a function to this and it will get called every time the frame changes.
  * It receives the `currentFrame` as a parameter, use it to call your functions at the exact frame!

## Ticker
_Trigger warning: some math ahead_

> Try to read a bit into the math before jumping into this one

```ts
import { Container, Sprite, Ticker } from "pixi.js";

export class Scene extends Container {
    private readonly screenWidth: number;
    private readonly screenHeight: number;

    private clampy: Sprite;
    private clampyVelocity: number = 5;
    constructor(screenWidth: number, screenHeight: number) {
        super();

        this.screenWidth = screenWidth;
        this.screenHeight = screenHeight;

        this.clampy = Sprite.from("clampy.png");

        this.clampy.anchor.set(0.5);
        this.clampy.x = 0; // we start it at 0
        this.clampy.y = this.screenHeight / 2;
        this.addChild(this.clampy);

        // See the `, this` thingy there? That is another way of binding the context!
        Ticker.shared.add(this.update, this);

        // If you want, you can do it the bind way
        // Ticker.shared.add(this.update.bind(this)); 
    }

    private update(deltaTime: number): void {
        this.clampy.x = this.clampy.x + this.clampyVelocity * deltaTime;

        if (this.clampy.x > this.screenWidth) {
            // Woah there clampy, come back inside the screen!
            this.clampy.x = 0;
        }
    }
}
```

Ok, PixiJS `Ticker` is an object that will call a function every frame before rendering and tell you how much time has passed since the last frame.  
But why is that useful? Well, when we know the velocity of something and we know for how long it has been moving we can predict where the object would be!  

In a silly example: If I know you can walk one block in 5 minutes, I know that if 10 minutes have passed you should be two blocks away.

Now, the actual spooky math is:

`New Position = Old Position + Velocity * Time Passed`

We can use this math every single frame to move an object after we give it a velocity! (Sorry if this was very obvious for you.)

Now that we are on board on what we need to do, let's see how to use the PixiJS Ticker.  
You can create one for yourself or just use the `Ticker.shared` instance that is already created for quick use. You just attach a function you want to be called every frame with the `.add()` and that's it!
[PixiJS Ticker API](https://pixijs.download/dev/docs/PIXI.Ticker_.html)  
Tips and Tricks:

* If you make your own `Ticker`, remember to `start()` it.
* The function you give in `.add(...)` can have a parameter time.
  * That parameter is a number in "how many frames passed at 60fps". So 1 means you are running at 60 fps. 2 means you are running at 30fps.
  * I personally hate it.
* `.deltaMS` and `.elapsedMS` will give you the amounts of milliseconds that passed between each call. `elapsedMS` is raw milliseconds while `deltaMS` can get affected by timescales or max framerates.
* `.FPS` good place to ask how many fps are you getting right now.

---

## Tweens

> We will use my own creation, Tweedle.js you can get it by `npm install tweedle.js`

```ts
import { Tween, Group } from "tweedle.js";
import { Container, Sprite, Ticker } from "pixi.js";

export class Scene extends Container {
    private clampy: Sprite;
    constructor(screenWidth: number, screenHeight: number) {
        super();

        this.clampy = Sprite.from("clampy.png");

        this.clampy.anchor.set(0.5);
        this.clampy.x = screenWidth / 2;
        this.clampy.y = screenHeight / 2;
        this.addChild(this.clampy);

        Ticker.shared.add(this.update, this);

        // See how these chains all together
        new Tween(this.clampy.scale).to({ x: 0.5, y: 0.5 }, 1000).repeat(Infinity).yoyo(true).start();

        // This is the same code, but unchained
        // const tweeny = new Tween(this.clampy.scale);
        // tweeny.to({ x: 0.5, y: 0.5 }, 1000);
        // tweeny.repeat(Infinity);
        // tweeny.yoyo(true);
        // tweeny.start();
    }

    private update(): void {
        //You need to update a group for the tweens to do something!
        Group.shared.update()
    }
}
```

> Note that you will still some ticker or loop to update the tweens

Ok, doing the math to move something with a given speed is fun and all, but what if I just want my element to do something in a certain amount of time and not bother to check if it already arrived, that it doesn't overshoot? To make it worse, what if I want some elastic moving movement?  

Tweens to the rescue! For tweens, I'll be using [Tweedle.js](https://github.com/miltoncandelero/tweedle.js) which is what I use every day in my job and it's my own fork of tween.js.  
Just like with PixiJS API, I won't copypaste the full API, feel free to check the [full Tweedle.js API Docs](https://miltoncandelero.github.io/tweedle.js/)  

Before we start, you need to remember **You need to update tweens too!**. This is usually achieved by updating the tween group they belong to. If you don't want to handle groups you can just use the shared static one that lives in `Group.shared`, just remember to update it.

Let's move through some of the basics:

* Create a tween with `new Tween(objectToChange);`
  * A second parameter can be passed if you want to use your custom groups, otherwise a shared static group that lives in `Group.shared` will be used.
* Tell your tween the target and duration with `.to( { property : targetValue }, duration)`
  * All durations of Tweedle are in **Milliseconds**
* Once you are ready to let it rip, just call `.start()`

Ok, but what other cool things can Tweedle do?

* Repeat your movement with `.repeat(amountOfTimesToRepeat)`
  * When repeating, use `.yoyo(true)` to have the tween reverse on each loop so it goes back and forth
* Spice up your movement with `.easing(Easing.Elastic.Out)`
  * There are a [lot of easings](https://miltoncandelero.github.io/tweedle.js/globals.html#easing) to pick!
* You can use bezier curves or tween between colors, it is some advanced stuff but it can be done. It's all in the [Interpolation](https://miltoncandelero.github.io/tweedle.js/globals.html#interpolation)

<aside class="warning">
All durations of Tweedle are in <b>Milliseconds</b>
</aside>

<aside class="warning">
Remember to <code>.update()</code> your groups!
</aside>