# Recipe: Lazy loading assets

Back in the day, people had patience to wait for a game or webpage to fully load before using it, but now users are a lot more impatient and want to have everything ready right now!  
To achieve this, the only solution is to have faster and faster internet connections but even that has a limit, so what can we do? **We fake it.**

By downloading just enough assets so that we can build the screen needed _right now_ we can hope to _bamboozle_ the user on a single screen while we keep loading the rest of the assets in the background and, hopefully, by the time they want to move on to the next screen we already have the assets.

What if we fail to _bamboozle_ the user long enough and they try to advance faster than what our assets download? Well, they will have to wait. It will be up to you to create and show them an _interstitial loading thingy_


## Initialize Early

> delete your `LoaderScene` and make some changes to the `Manager.initialize()`

```ts
import { Assets } from "pixi.js";
import { manifest } from "../assets";

export class Manager {
    // ...

    // This is a promise that will resolve when Assets has been initialized.
    // Promise<unknown> means this is a promise but we don't care what value it resolves to, only that it resolves.
    private static initializeAssetsPromise: Promise<unknown>; 

    // ...

    public static initialize(width: number, height: number, background: number): void {
        
        // We store it to be sure we can use Assets later on
        Manager.initializeAssetsPromise = Assets.init({ manifest: manifest });

        // Black js magic to extract the bundle names into an array.
        const bundleNames = manifest.bundles.map(b => b.name);

        // Initialize the assets and then start downloading the bundles in the background
        Manager.initializeAssetsPromise.then(() => Assets.backgroundLoadBundle(bundleNames));

        // ...
    }
    // ...
}
```

> Bonus points if you want to make initialize `async` and `await` the initialize promise...

For this setup, we will no longer use the `LoaderScene`, so we need to make sure we initialize `Assets` in our `Manager` constructor before we create any `Scene`.  
We also set to download everything in the background. Whenever we need to load something for a scene, the background downloads will pause and then resume _automagically._

## Bundles we depend on

> we make a variable to store what bundles we need

```ts
export interface IScene extends DisplayObject {
    // ...
    assetBundles:string[];
    // ...
}
```

> make sure you write something inside that array inside your `Scene`!

```ts
export class GameScene extends Container implements IScene {
    // ...
    assetBundles:string[] = ["game", "sounds"];
    // ...
}
```

For this to work, we will need a way for each `Scene` to know which _asset bundles_ it needs to show itself. Depending on how we divided our assets we might need just one bundle per scene or we might need many. We will use an array just in case.

We can also set the scene constructor to begin downloading our _asset bundles_ but...

## Async constructors aren't a thing!

> we **can't** do this!

```ts
export class GameScene extends Container implements IScene {
    
    assetBundles:string[] = ["game", "sounds"];

    constructor() {
        super();

        // This is illegal!
        await Assets.loadBundle(this.assetBundles);

        // But we need to wait to have assets D:
        const clampy = Sprite.from("Clampy the clamp!");
        this.addChild(clampy);
    }

    update(framesPassed: number): void {}

}
```

> We go in a different way, let Manager load your assets and let the scene now.

```ts
export interface IScene extends DisplayObject {
    // ...
    assetBundles: string[];
    constructorWithAssets(): void;
    // ...
}
```

> And change how Manager changes the scene...

```ts
export class Manager {
    
    // ...
    
    public static async changeScene(newScene: IScene): Promise<void> {

        // let's make sure our Assets were initialized correctly
        await Manager.initializeAssetsPromise;


        // Remove and destroy old scene... if we had one..
        if (Manager.currentScene) {
            Manager.app.stage.removeChild(Manager.currentScene);
            Manager.currentScene.destroy();
        }

        // If you were to show a loading thingy, this will be the place to show it...

        // Now, let's start downloading the assets we need and wait for them...
        await Assets.loadBundle(newScene.assetBundles);

        // If you have shown a loading thingy, this will be the place to hide it...

        // when we have assets, we tell that scene
        newScene.constructorWithAssets();

        // we now store it and show it, as it is completely created
        Manager.currentScene = newScene;
        Manager.app.stage.addChild(Manager.currentScene);
    }

    // ...

}
```

> Finally, an example of how an scene should kinda look

```ts
export class GameScene extends Container implements IScene {
    
    assetBundles:string[] = ["game", "sounds"];

    constructor() {
        super();
        // We can make anything here as long as we don't need assets!
    }
    constructorWithAssets(): void {
        // Manager will call this when we have all the assets!

        const clampy = Sprite.from("Clampy the clamp!");
        this.addChild(clampy);
    }
    update(framesPassed: number): void {}
}
```

As I said before, `async` constructors are not a thing so we can't `await` on the construction of a `Scene` for our assets. We are forced to use a method to finish the construction of our `Scene`. Let's say this is the `constructorWithAssets()` method. We will delegate the responsibility of downloading the assets a `Scene` needs to the `Manager` and make it tell the scene when the assets are downloaded.

<aside class="info">Remember, we fired <code>Assets.backgroundLoadBundle(...)</code> before, so hopefully the assets were already downloaded and the <code>Assets.loadBundle(...)</code> call should resolve immediately.</aside>

It is left as an exercise to the reader to make the _interstitial loading thingy_ in case the user goes too fast and has to wait for more assets to download.  
(Or you can cheat and always keep the _interstitial loading thingy_ in the background and it will only be seen when there is no scene covering it)