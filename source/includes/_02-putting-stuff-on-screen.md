# Putting stuff on screen
_(and being in control of said stuff)_

##  The DisplayList
_In my not-so-humble opinion, the best way to handle 2d graphics._  

It all starts with an abstract class: The `DisplayObject`. Anything that can be shown on the screen **must** inherit from this abstract class at some point in his genealogy. When I want to refer to "anything that can be placed on the screen" I will use the generic term `DisplayObject`.  
In PixiJS your bread and butter are going to be `Container` and `Sprite`. _Sprites_ can show graphics (if you come from the [getting started](#getting-started) you've already seen it in action) and _Containers_ are used to group sprites to move, rotate and scale them as a whole. You can add a _Container_ to another _Container_ and keep going as deep as you need your rabbit hole to go.  
Imagine a cork bulletin board, some photos, and a box of thumbtacks. You could pin every photo directly to the board... or you could pin some photos to another and then move them all together by moving the photo you pinned every other photo to.  
In this relationship, the _Container_ is called the **parent** and the _DisplayObjects_ that are attached to this parent are called **children**.
Things are rendered _back-to-front_, which means that children will always be covering their parents. Between siblings (children with the same parent), the render order will start from the first added child and move to the last one, making the last child added show on top of all his siblings (unless you use the `zOrder` property of any display object. However, this property only works between siblings. A child will never be able to be behind his parent).  
I know that it has **list** in the name, but it actually looks more like a tree. If you are a nerd like me and understand data structures, imagine a tree structure where each node can have any number of nodes attached that can have more nodes attached...  
Finally, we have the root of everything, the granddaddy of them all, the greatest of grandfathers, and we shall call it the `Stage`. The stage is just a regular container that the `Application` class creates for us and feeds it to the `Renderer` to... well render it.

### Have some code

```ts
import { Application, Sprite, Container } from 'pixi.js'

const app = new Application({
	view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
	resolution: window.devicePixelRatio || 1,
  autoDensity: true,
	backgroundColor: 0x6495ed,
	width: 640,
	height: 480
});

const conty: Container = new Container();
conty.x = 200;
conty.y = 0;
app.stage.addChild(conty);

const clampy: Sprite = Sprite.from("clampy.png");
clampy.x = 100;
clampy.y = 100;
conty.addChild(clampy);
```

Ok, let's make a simple example to test `Container` and `Sprite`.  

It should look fairly familiar, but let's go real quick over it...  
We create _conty_ the `Container` and add it to the `app.stage`. That means that it is added to the screen and should get rendered but it is empty...  
So we create _clampy_ the clamp `Sprite` like the last time but instead of adding it directly to the screen, we add it to _conty_ the `Container`.  
Now, see how we set Clampy's position to 100, 100 but if you run it, you will see that it is not showing up there but way more to the side... why?  
Well, we added Clampy to _conty_ and he has an `x` value of 200. That means that Clampy now has a _global_ `x` value of 300!

#### Homework!
Ok, let's try some things and see what happens!

* Try making more Containers and setting different positions.
  * Try to be able to predict where the sprite will end up by keeping a mental map of all the parents and grandparents it has.
* Try adding the same sprite twice to two different containers... Did it work?
  * Spoiler alert, it won't work. A _DisplayObject_ can't have two parents, it will become unattached from his current parent to go to the new one.
  * Make more sprites with `Sprite.from()` to test multiple sprites on screen.
* Try using a `Sprite` as a `Container`.
  * This works because `Sprite` actually inherits from `Container`!
* Try rotations and scales on parents. See how the origin of the parent becomes relevant to the position of the children?
  * Try to make a sprite rotate around his center without using `origin` or `anchor` but instead centering it on the origin of his parent and rotating his parent.
    * Fun fact: in ye' old times this was the only way to have a sprite rotate around his center!

## A quick overview of the Display Objects you can use out of the box.
_Note: I will just give you a quick overview and a link to the official PixiJS API, if you want to know everything about an object you should got here._

## Container

> an example where `bigConty` is the papa of `littleConty`.

```ts
const bigConty: Container = new Container();
bigConty.scale.set(2); // You can use set and only one value to set x and y
bigConty.position.x = 100;
bigConty.y = 200; // this is a shortcut for .position.y and it also exists one for .position.x
app.stage.addChild(bigConty);

const littleConty: Container = new Container();
// position has a copy setter. It won't use your reference but copy the values from it!
littleConty.position = new Point(300,200);
bigConty.addChild(littleConty);
```

> A bit of a silly example, it won't show anything on screen since containers don't have content by themselves.

The most basic class you will have. While it doesn't have any graphical representation on its own, you can use it to group other objects and affect them as a unit.  
[PixiJS Container API](https://pixijs.download/dev/docs/PIXI.Container.html)  
Methods and properties that you will use most frequently:  

* `addChild(...)`: You use this to add something to the container.
* `removeChild(...)`: You use this to remove said something to the container.
* `children`: An array of the objects you added to the container.
* `position`, `scale` and `skew`: If you care to know, those are two PixiJS [Point](https://pixijs.download/dev/docs/PIXI.ObservablePoint.html)
  * `position.x` and `position.y` have shortcuts on `.x` and `.y`
  * Even if `position` is an object, setting it to a new position or to another object's position won't break it since it has a setter that copies the value! This makes `one.position = another.position` totally safe!
* `rotation` and `angle`: Rotation is in _radians_ while angle is in _degrees_. Changing one updates the other. 
* `width` and `height`: The size of a container it's defined by the size of a box that contains all his children. This means it changes if the children move. 
  * **Changing these values modifies the scale of the object.**
* `interactive`, `on(...)` and `off(...)`: With this, you can manage your event listeners. It will be really useful when we get to see [interacting with your display objects](#getting-interactive)
* `getBounds(...)`: Will give you a rectangle of this object. It will be very useful for [detecting basic collisions](#collision-detection).
* `destroy()` This will remove the object from his parent and render it unusable forever. After that, get rid of any reference and the garbage collector should eat it.

### Particle Container

> There isn't much to them, it's like a regular container

```ts
const particleConty: ParticleContainer = new ParticleContainer();
// Pretty much everything that worked on a Container will work with a ParticleContainer.
```

A [Particle Container](https://pixijs.download/dev/docs/PIXI.ParticleContainer.html) is a special kind of container designed to go fast. To achieve this extra speed you sacrifice some functionality.  
The rules for Particle Containers are:

* **No grandchildren**: Children of a ParticleContainer can't have children.
* **No fancy stuff**: Filters and Masks won't work here.
* **Single texture**: All Sprites inside a ParticleContainer **must** come from the same texture. If you are using texture atlases this is easier to achieve. If you ever see one of your sprites inside a ParticleContainer looking like a garbled mess of another sprite it is probably because they were not sharing a texture.

While the name seems to indicate that _Particle Containers_ should be used for _Particles_ you can use them as super fast containers when you need to render a lot of objects.

<aside class="notice">
If you check the PixiJS documentation for ParticleContainer you will see no mention of the <b>single texture</b> rule. However, trust me that this rule is true... or try to prove me wrong and see how your sprites become garbled messes.
</aside>

## Sprite

> The simple example from the getting started

```ts
const clampy: Sprite = Sprite.from("clampy.png");

clampy.anchor.set(0.5);

// setting it to "the middle of the screen
clampy.x = app.screen.width / 2;
clampy.y = app.screen.height / 2;

app.stage.addChild(clampy);
```

> Here we use again the shortcuts for `position`. 

The simplest way to show a bitmap on your screen. It inherits from `Container` so all the properties from above apply here too!  
[PixiJS Sprite API](https://pixijs.download/dev/docs/PIXI.Sprite.html)  
Methods and properties that you will use most frequently:  

* `Sprite.from(...)`: This is a static method to create new sprites. It does some black magic inside it so that it can take a lot of different kinds of parameters. (You technically can use the `new Sprite(...)` way of creating sprites but this is way easier).  
This method can take any of the following parameters:
  * Name of a texture you loaded before (we will see [how to load assets in another chapter](#recipe-preloading-assets)).
  * URL of an image or video. This can be an absolute one (`http://somedomain.com/image.png`) or relative (`assets/image.png`).
  * A PixiJS Base texture
  * A canvas or video HTML element
* `anchor`: This allows you to set where you want the center of this sprite to be. `0, 0` sets the origin to the left and top, and `1, 1` means the right and bottom.
* `tint`: Fast way to color a sprite. The default value is white `0xFFFFFF` and this means no color change. **This is done inside the shader and is F R E E. No performance hit whatsoever!**

## Graphics

> There is SO much stuff you can do with graphics... Let's just make a circle at 100,100

```ts
const graphy: Graphics = new Graphics();

// we give instructions in order. begin fill, line style, draw circle, end filling
graphy.beginFill(0xFF00FF);
graphy.lineStyle(10, 0x00FF00);
graphy.drawCircle(0, 0, 25); // See how I set the drawing at 0,0? NOT AT 100, 100!
graphy.endFill();

app.stage.addChild(graphy); //I can add it before setting position, nothing bad will happen.

// Here we set it at 100,100
graphy.x = 100;
graphy.y = 100;
```

> I can't stress this enough: **Do** draw your graphics relative to their own origin and then move the object. **Don't** try to draw it directly on the screen position you want

This class allows you to make primitive drawings like rectangles, circles, and lines. It is really useful when you need masks, hitboxes, or want a simple graphic without needing a bitmap file. It also inherits from `Container`.  
[PixiJS Graphics API](https://pixijs.download/dev/docs/PIXI.Graphics.html)  
Methods and properties that you will use most frequently:  

* `beginFill(...)` and `endFill()`: You mark the beginning and end of a fill. Every shape you draw between the _begin_ and _end_ calls will be filled by the color you specify.
* `lineStyle(...)`: Defines color, thickness, and other properties of the line of your drawings.
* `moveTo(...)`, `lineTo(...)`, `drawRect(...)` and `drawCircle(...)`: The most basics tools for drawing. There are more complex ones like polygons and beziers! Check them out in the API.
* `clear()`: for when you need to erase everything and start over.

<aside class="warning">
A final note for graphics, because I've seen some... stuff: Don't try to <code>clear()</code> and then redraw all your <code>Graphics</code> every frame when you need to move it! You should ideally draw them once and then move, (scale, rotate, etc) the resulting object.
</aside>

## Text

> Check [PixiJS Textstyle Editor](https://pixijs.io/pixi-text-style) to make the textstyle easily.

```ts
const styly: TextStyle = new TextStyle({
    align: "center",
    fill: "#754c24",
    fontSize: 42
});
const texty: Text = new Text('私に気づいて先輩！', styly); // Text supports unicode!
texty.text = "This is expensive to change, please do not abuse";

app.stage.addChild(texty);
```

> (Japanese text is optional, I used it just to show Unicode support)

Oh boy, we could have an entire chapter dedicated to text but for now, just the basics.  
Text has AMAZING support for Unicode characters (as long as your chosen font supports it) and it is pretty consistent on how it looks across browsers.  
[PixiJS Text API](https://pixijs.download/dev/docs/PIXI.Text.html)  
Tips:  

* Go to [PixiJS Textstyle Editor](https://pixijs.io/pixi-text-style) to make your text look exactly like you want it to.
* `text`: Contains the text to show. **Changing this is expensive**. If you need your text to change every frame (for example, a score) consider using `BitmapText`
* To use custom fonts you need to add them as webfonts to your webpage. If you know your _html-fu_ you can do this or you can check the [fonts section on how to load assets](#recipe-preloading-assets)

## BitmapText

> [PixiJS Textstyle Editor](https://pixijs.io/pixi-text-style) can be used to make the object for the `BitmapFont.from(...)` thingy.

```ts
// If you need to know, this is the expensive part. This creates the font atlas
BitmapFont.from("comic 32", {
	fill: "#ffffff", // White, will be colored later
	fontFamily: "Comic Sans MS",
	fontSize: 32
})

// Remember, this font only has letters and numbers. No commas or any other symbol.
const bitmapTexty: BitmapText = new BitmapText("I love baking, my family, and my friends",
	{
		fontName: "comic 32",
		fontSize: 32, // Making it too big or too small will look bad
		tint: 0xFF0000 // Here we make it red.
	});

bitmapTexty.text = "This is cheap";
bitmapTexty.text = "Change it as much as you want";

app.stage.addChild(bitmapTexty);
```

> Remember, symbols won't show by default. Your sentence might not mean the same without commas.

The faster but more limited brother to `Text`, BitmapText uses a BitmapFont to draw your text. That means that changing your text is just changing what sprites are shown on screen, which is really fast. Its downside is that if you need full Unicode support (Chinese, Japanese, Arabic, Russian, etc) you will end up with a font atlas so big that performance will start to go down again.  
[PixiJS BitmapText API](https://pixijs.download/dev/docs/PIXI.BitmapText.html)  
Tips:  

* **PixiJS can create a BitmapFont on the fly from a regular font.** `BitmapFont.from(...)` will take an object similar to a TextStyle and make a font for you to use!
  * You will give this generated font a `name` and that will be the unique identifier. Creating another font with the same name will overwrite that style!
  * I advise making the BitmapFont in white as you can tint it later with the BitmapText object.
  * When creating a font this way, you can pass a custom set of characters to include in the BitmapFont atlas. `BitmapFont.ALPHA`, `BitmapFont.NUMERIC`, and `BitmapFont.ALPHANUMERIC` are constants you can use or you can create your own string.
* The screen size of the BitmapText is achieved by scaling the BitmapFont. This can lead to ugly-looking text if you are pushing your sizes too far apart. Try creating fonts that are closer in size to your needs.

## Filters
_Stunning effects with no effort!_

PixiJS has a stunning collection of filters and effects you can apply either to only one _DisplayObject_ or to any _Container_ and it will apply to all its children!
I won't cover **all** the filters (at the time of writing there are 37 of them!!) instead, I will show you how to use one of the pre-packaged filters and you will have to extrapolate the knowledge from there.  
[You can see a demo of the filters](http://filters.pixijs.download/dev/demo/index.html) or [go directly to Github to see what package to install](https://github.com/pixijs/filters).  

> Creating and using filters is so easy that I wasn't sure if I needed to make this part or not

```ts
// import the filters
// If you are using pixijs < 6 you might need to import `filters`
import { BlurFilter } from "pixi.js"; 

// Make your filter
const myBlurFilter = new BlurFilter();

// Add it to the `.filters` array of any DisplayObject
clampy.filters = [myBlurFilter];
```

This is a section that I almost didn't make because using filters is super simple but I made it anyway so you guys know there is a huge list of filters ready to use.

In essence, create a filter, add it to the filters array of your display object, and you are done!

<aside class="notice">
Technically applying a filter on a <code>Sprite</code> should be cheaper than applying it to a <code>Container</code> but I haven't made enough tests to be able to tell you for certain.
</aside>

## Particles
_Make it rain_

> Make sure you dropped your emitter.json somewhere inside your src folder

```ts
import * as particleSettings from "../emitter.json";


const particleContainer = new ParticleContainer();
app.stage.addChild(particleContainer);

const emitter = new particles.Emitter(particleContainer, Texture.from("particleTexture.png"), particleSettings);
emitter.autoUpdate = true; // If you keep it false, you have to update your particles yourself.
emitter.updateSpawnPos(200, 100);
emitter.emit = true;
```


For handling particles, PixiJS uses their own format and you need to install the library: `npm install pixi-particles` and you can create them using [the Particle Designer](https://pixijs.io/pixi-particles-editor/)

Once we have our particles looking good on the editor we download a `.json` file and we will feed the parsed object into the `Emitter` constructor along with a container for our particles (it can be any `Container` but if you have a lot of particles you might want a `ParticleContainer`), and the `Texture` (or an array of textures) you want your particles to use.

To make that `.json` into something usable we need to either load it with the `Loader` or just add it as part of our source code and `import` it.

Methods and properties that you will use most frequently:  

* `playOnce(...)`: Starts emitting particles and optionally calls a callback when particle emission is complete.
* `playOnceAndDestroy(...)`: Starts emitting particles, sets autoUpdate to true, and sets up the Emitter to destroy itself when particle emission is complete.
* `autoUpdate`: Enable or disable the automatic update.
* `update(seconds)`: If autoUpdate is false, you need to manually move the time forward. This method wants the time in **seconds**.
* `updateSpawnPos(...)`: Move your emitter around
* `resetPositionTracking()`: When you move your emitter, it will leave a trail behind itself. If you want to teleport it without a trail, call this method after setting the spawn position.
