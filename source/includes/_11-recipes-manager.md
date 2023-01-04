# Recipe: Scene Manager

In the [Splitting Code](#splitting-code) chapter I explained how to create a `Scene` object to try to encapsulate different parts of our object and be able to swap them when needed but I didn't explain how to actually change from one scene to the next.  
For this purpose, we are going to create a _Manager_ static global class that wraps the PixiJS `Application` object and exposes a simple way to change from one scene to the next.

<aside class="info">If you are a hardcore programmer from another language you might be familiar with the Singleton pattern and might prefer to use that instead of a static class. Feel free to do so if you feel that is better.</aside>

## The Manager Class

> Ok, let's write our `Manager.ts` file. This file will store the static class _Manager_ and an Interface for our _Scenes_

```ts
import { Application, DisplayObject } from "pixi.js";

export class Manager {
    private constructor() { /*this class is purely static. No constructor to see here*/ }

    // Safely store variables for our game
    private static app: Application;
    private static currentScene: IScene;

    // Width and Height are read-only after creation (for now)
    private static _width: number;
    private static _height: number;


    // With getters but not setters, these variables become read-only
    public static get width(): number {
        return Manager._width;
    }
    public static get height(): number {
        return Manager._height;
    }

    // Use this function ONCE to start the entire machinery
    public static initialize(width: number, height: number, background: number): void {

        // store our width and height
        Manager._width = width;
        Manager._height = height;

        // Create our pixi app
        Manager.app = new Application({
            view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
            resolution: window.devicePixelRatio || 1,
            autoDensity: true,
            backgroundColor: background,
            width: width,
            height: height
        });

        // Add the ticker
        Manager.app.ticker.add(Manager.update)
    }

    // Call this function when you want to go to a new scene
    public static changeScene(newScene: IScene): void {
        // Remove and destroy old scene... if we had one..
        if (Manager.currentScene) {
            Manager.app.stage.removeChild(Manager.currentScene);
            Manager.currentScene.destroy();
        }

        // Add the new one
        Manager.currentScene = newScene;
        Manager.app.stage.addChild(Manager.currentScene);
    }

    // This update will be called by a pixi ticker and tell the scene that a tick happened
    private static update(framesPassed: number): void {
        // Let the current scene know that we updated it...
        // Just for funzies, sanity check that it exists first.
        if (Manager.currentScene) {
            Manager.currentScene.update(framesPassed);
        }

        // as I said before, I HATE the "frame passed" approach. I would rather use `Manager.app.ticker.deltaMS`
    }
}

// This could have a lot more generic functions that you force all your scenes to have. Update is just an example.
// Also, this could be in its own file...
export interface IScene extends DisplayObject {
    update(framesPassed: number): void;
}
```

Lots to unpack here but let's take it easy.  
First of all, see that since I made this a fully static class I never use `this` object but instead I refer to everything by using the `Manager.` notation. This prevents the need to keep track of the context in this class.  

Next, we have a private constructor; that means nobody will be able to do `new Manager()`. That is important because our class is fully static and it's meant to be initialized by calling `Manager.initialize(...)` and we feed parameters about the screen size and the background color for our game.  
The `Application` instance and the _width_ and _height_ are safely stored and hidden in private static variables but these last two have getters so we can ask globally what is the size of our game.  

The initialize function has one last trick: it adds links from your entire `Manager` to a ticker so that our `Manager.update()` method gets called. That update will then call the current scene update method. This will allow you to have an update method in your scenes that appears to be called _magically_.

Then we have the `Manager.changeScene(...)` method, this is exactly why we started all of this, an easy way to tell our game that the current scene is no longer useful and that we want to change it for a new one.  
It's quite simple to see that `Manager` removes the old one from the screen, destroys it (actually, `destroy()` is a PixiJS method!), and then proceeds to put your new scene directly on screen.  

But...  
_What is a Scene? A miserable little pile of pixels!_  
Well, it has to be a `DisplayObject` of some sort because we need to put it on the screen (remember that `Containers` _are DisplayObjects_) but we also want it to have our custom `update(...)` method and to enforce that we create an interface `IScene` (I like to begin my interfaces with `I`). This makes sure that if an object wants to be a _Scene_ then it must follow the two rules: be some sort of `DisplayObject` and have an `update(...)` method.

## Now, some reruns of your favorite classes!

> Look at this buffed up `LoaderScene.ts` that now `implements IScene` and uses `Manager` to know its size!

```ts
import { Container, Graphics, Assets } from "pixi.js";
import { manifest } from "../assets";
import { IScene, Manager } from "../Manager";
import { GameScene } from "./GameScene";

export class LoaderScene extends Container implements IScene {

    // for making our loader graphics...
    private loaderBar: Container;
    private loaderBarBoder: Graphics;
    private loaderBarFill: Graphics;
    constructor() {
        super();

        const loaderBarWidth = Manager.width * 0.8;

        this.loaderBarFill = new Graphics();
        this.loaderBarFill.beginFill(0x008800, 1)
        this.loaderBarFill.drawRect(0, 0, loaderBarWidth, 50);
        this.loaderBarFill.endFill();
        this.loaderBarFill.scale.x = 0;

        this.loaderBarBoder = new Graphics();
        this.loaderBarBoder.lineStyle(10, 0x0, 1);
        this.loaderBarBoder.drawRect(0, 0, loaderBarWidth, 50);

        this.loaderBar = new Container();
        this.loaderBar.addChild(this.loaderBarFill);
        this.loaderBar.addChild(this.loaderBarBoder);
        this.loaderBar.position.x = (Manager.width - this.loaderBar.width) / 2;
        this.loaderBar.position.y = (Manager.height - this.loaderBar.height) / 2;
        this.addChild(this.loaderBar);

        this.initializeLoader().then(() => {
            this.gameLoaded();
        })
    }

    private async initializeLoader(): Promise<void>
    {
        await Assets.init({ manifest: manifest });

        const bundleIds =  manifest.bundles.map(bundle => bundle.name);

        await Assets.loadBundle(bundleIds, this.downloadProgress.bind(this));
    }

    private downloadProgress(progressRatio: number): void {
        this.loaderBarFill.scale.x = progressRatio;
    }

    private gameLoaded(): void {
        // Change scene to the game scene!
        Manager.changeScene(new GameScene());
    }

    public update(framesPassed: number): void {
        // To be a scene we must have the update method even if we don't use it.
    }
}
```
In this new `LoaderScene` that extends from `IScene` there are really only two changes:  
* It now extends from `IScene` so it must have an `update(...)` method even if we don't need it.
* When the loader finishes, it changes the scene to another one! We can finally see `Manager.changeScene()` in action!

> And this is what the simpler `GameScene.ts` looks like.

```ts
import { Container, Sprite } from "pixi.js";
import { IScene, Manager } from "../Manager";

export class GameScene extends Container implements IScene {
    private clampy: Sprite;
    private clampyVelocity: number;
    constructor() {
        super();

        // Inside assets.ts we have a line that says `"Clampy from assets.ts!": "./clampy.png",`
        this.clampy = Sprite.from("Clampy from assets.ts!");

        this.clampy.anchor.set(0.5);
        this.clampy.x = Manager.width / 2;
        this.clampy.y = Manager.height / 2;
        this.addChild(this.clampy);

        this.clampyVelocity = 5;
    }
    public update(framesPassed: number): void {
        // Lets move clampy!
        this.clampy.x += this.clampyVelocity * framesPassed;

        if (this.clampy.x > Manager.width) {
            this.clampy.x = Manager.width;
            this.clampyVelocity = -this.clampyVelocity;
        }

        if (this.clampy.x < 0) {
            this.clampy.x = 0;
            this.clampyVelocity = -this.clampyVelocity;
        }
    }
}
```

## Turning on the entire machine

> `index.ts` has always been the entry point of our game. Let's use it to start our `Manager` and open our first ever `Scene`

```ts
import { Manager } from './Manager';
import { LoaderScene } from './scenes/LoaderScene';

Manager.initialize(640, 480, 0x6495ed);

// We no longer need to tell the scene the size because we can ask Manager!
const loady: LoaderScene = new LoaderScene();
Manager.changeScene(loady);
```

So far we have a _Manager_ that can change from one _Scene_ to the next but... how do we start it and go to the first scene ever?  
That is what the `Manager.initialize(...)` was for and we will call it directly from `index.ts` and right after that, we ask it to go to the first scene of our game: the `LoaderScene`.  

That is all there is to it. You _initialize_ your _manager_ and are ready to rumble from _scene_ to _scene_!