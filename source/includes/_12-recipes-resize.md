# Recipe: Resize your game

By now you probably have realized that all the examples so far have the game in the top-left corner of the screen, this is because we never bothered to move it or change its size: Let's make it fill the screen.  
There are two ways to fill the screen that I like to call _letterbox_ and _responsive_.  
In _letterbox_ you scale your entire game and add black bars to the side when the aspect ratio doesn't match, this is the easier approach and is good enough for small games that will be embedded in an iframe of another website. By using this method the resize is done entirely by the browser with some _css_ trickery so you don't have to add any logic inside your game.
In _responsive_ you make your `stage` grow to fill the entire screen and you have to resize and rearrange your game elements to make your game take advantage of all the screen space available, this is almost is mandatory if you are trying to make a game that looks and feel _native_ on every device (mostly smartphones). However, this approach is _harder_ to implement as you have to always be mindful of the scale of your objects, sizes, and positions of the elements of your game.

> To ask the browser what is the current screen size we will use a small catch-all piece of code. This is to make sure we get the correct measurement no matter what web browser the user has.

```ts
const screenWidth = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
const screenHeight = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);
```

In any case, we will know the initial screen size (and if the screen changed the size) by using what the browser provides us.

## Letterbox scale

> Here you have a modified `Manager.ts` class that listens for a resize event and uses css to fix the size of the game.

```ts
export class Manager {
    private constructor() { }

    private static app: Application;
    private static currentScene: IScene;

    private static _width: number;
    private static _height: number;


    public static get width(): number {
        return Manager._width;
    }
    public static get height(): number {
        return Manager._height;
    }


    public static initialize(width: number, height: number, background: number): void {

        Manager._width = width;
        Manager._height = height;

        Manager.app = new Application<HTMLCanvasElement>({
            view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
            resolution: window.devicePixelRatio || 1,
            autoDensity: true,
            backgroundColor: background,
            width: width,
            height: height
        });

        Manager.app.ticker.add(Manager.update)

        // listen for the browser telling us that the screen size changed
        window.addEventListener("resize", Manager.resize);

        // call it manually once so we are sure we are the correct size after starting
        Manager.resize();
    }

    public static resize(): void {
        // current screen size
        const screenWidth = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
        const screenHeight = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);

        // uniform scale for our game
        const scale = Math.min(screenWidth / Manager.width, screenHeight / Manager.height);

        // the "uniformly englarged" size for our game
        const enlargedWidth = Math.floor(scale * Manager.width);
        const enlargedHeight = Math.floor(scale * Manager.height);

        // margins for centering our game
        const horizontalMargin = (screenWidth - enlargedWidth) / 2;
        const verticalMargin = (screenHeight - enlargedHeight) / 2;

        // now we use css trickery to set the sizes and margins
        Manager.app.view.style.width = `${enlargedWidth}px`;
        Manager.app.view.style.height = `${enlargedHeight}px`;
        Manager.app.view.style.marginLeft = Manager.app.view.style.marginRight = `${horizontalMargin}px`;
        Manager.app.view.style.marginTop = Manager.app.view.style.marginBottom = `${verticalMargin}px`;
    }

    /* More code of your Manager.ts like `changeScene` and `update`*/
}
```

> If you are not using a _Manager_ you need to focus on the `window.addEventListener(...)` and the `resize()` method.

Letterbox scale implies two things: Making our game _as big as possible_ (while still fitting on screen) and then _centering it on the screen_ (letting black bars appear on the edges where the ratio doesn't match).  
It is very important that when we make it _as big as possible_ we don't stretch it and make it look funky, to make sure of this we find the _enlargement factor_ and increase our width and height by multiplying this _factor_.  
To get a factor you just _divide the size of the screen by the size of your game_ but since you have _width_ and _height_ you end up with two factors. To make sure our game doesn't _bleed out_ of the screen and _fits inside_ we pick the smaller of the two factors we found and we multiply our _width_ and _height_ by it.

After we have our _englarged_ size by multiplying by a factor, we calculate the difference between that size and the size of the screen and split it evenly in the margins so our game ends up centered.

<aside class="info">
You might see that I read but never change the <code>Manager.width</code> and <code>Manager.height</code> variables. That is because our game never realizes it is now being enlarged: everything happens at the browser level with css so our game still believes to have the size that <code>Manager.width</code> and <code>Manager.height</code> says.
</aside>

## Responsive Scale

> Here you have a modified `Manager.ts` class that listens for a resize event and notifies the current scene of the new size.

```ts

export class Manager {
    private constructor() { }
    private static app: Application;
    private static currentScene: IScene;

    // We no longer need to store width and height since now it is literally the size of the screen.
    // We just modify our getters
    public static get width(): number {
        return Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
    }
    public static get height(): number {
        return Math.max(document.documentElement.clientHeight, window.innerHeight || 0);
    }

    public static initialize(background: number): void {

        Manager.app = new Application<HTMLCanvasElement>({
            view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
            resizeTo: window, // This line here handles the actual resize!
            resolution: window.devicePixelRatio || 1,
            autoDensity: true,
            backgroundColor: background,
        });

        Manager.app.ticker.add(Manager.update)

        // listen for the browser telling us that the screen size changed
        window.addEventListener("resize", Manager.resize);
    }

    public static resize(): void {
        // if we have a scene, we let it know that a resize happened!
        if (Manager.currentScene) {
            Manager.currentScene.resize(Manager.width, Manager.height);
        }
    }

    /* More code of your Manager.ts like `changeScene` and `update`*/
}

export interface IScene extends DisplayObject {
    update(framesPassed: number): void;

    // we added the resize method to the interface
    resize(screenWidth: number, screenHeight: number): void;
}
```

> If you are not using a _Manager_ you need to focus on the `resizeTo: window,` line in the constructor of `Application`.

To turn on Responsive resize is actually just one line of code: you add `resizeTo: window,` to your PixiJS `Application` object and it will work however your game won't realize that now has to move and fill the new space (or fit inside a smaller space).  
To make this task a bit less hard, we need to know _when_ the game changed size and what is that new size.  
To do this, we will listen for the `resize` event, update our `Manager.width` and `Manager.height` variables, and let the current scene know that a change in size occurred. To _let the scene know_ we are going to add a function to our `IScene` interface so all scenes must now have the `resize(...)` method.  

After that... you are on your own: You need to write your own code inside each scene `resize(...)` method to make sure every single object in your game reacts accordingly to your new screen size. Good luck.

<aside class="info">
You might realize that we no longer have `width` and `height` variables but instead, we calculate them directly from the screen. You might want to store an <i>ideal size</i> for making scale calculations.
</aside>

## Resolution (Device Pixel Ratio)

```ts
new Application<HTMLCanvasElement>({
    view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
    resolution: window.devicePixelRatio || 1, // This bad boy right here...
    autoDensity: true, // and his friend
    backgroundColor: background,
    width: width,
    height: height
});
```

The keen-eyed of you might have noticed that when we create our `Application` object we have a line that says `resolution: window.devicePixelRatio || 1` but what does that mean?  
The standard for screens was set to 96 dpi (dots per inch) for a long time and we were happy until the Apple nation attacked with their _Retina Display_ that had twice as many dots per inch, thus if `devicePixelRatio` was equal to 1 it meant _96 dpi_ and if it was equal to 2 it was _Retina display_.  
And then the world kept moving forward, adding more and more dots per inch, in seemingly random amounts so `devicePixelRatio` started reporting decimal numbers left and right: everything as a factor of that original  _96 dpi_.  

There is the complementary line that says `autoDensity: true` and that is meant for the `InteractionManager`. It makes it so that DOM and CSS coordinates (and thus, touch and mouse coordinates) match your global pixel units.

By letting feeding the `Apllication` constructor the `devicePixelRatio` we render our game in a _native resolution_ for the displays dpi, resulting in a sharper image on devices that have more than _96 dpi_ but at the cost of some performance since we are effectively _supersampling_ every pixel.  

If a particular device (most likely an iOS device) isn't strong enough to render all the pixels of the _native resoluton_ you can always force the resolution to 96 dpi with `resolution: 1` and get an image that might be blurry but you get a non-despicable performance boost.

<aside class="info">
Let's get the javascript out of the way: <code>window.devicePixelRatio</code> should always be a number however some old browsers might say <code>NaN</code> or <code>undefined</code> and feeding such a value to the <code>Application</code> constructor might produce unexpected results. That <code>||1</code> makes sure we never feed it something that isn't a number.
</aside>

<aside class="info">
Fun fact: Some devices consider the standard to be 76 dpi instead of 96. However don't worry, this doesn't change how we use <code>devicePixelRatio</code>
</aside>