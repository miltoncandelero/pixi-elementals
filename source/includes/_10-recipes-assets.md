# Recipe: Preloading assets

So far we have seen how to create images and sounds by downloading the asset behind them just as we need to show it to the user. While this is good enough if we want a quick way to make a proof of concept or prototype, it won't be good enough for a project release.  
The old and elegant way of doing it is downloading all the assets you are going to need beforehand and storing them in some sort of cache.   
However, times change and users now want everything ready **right now!** so instead of having one big load at the beginning we will aim to have the smaller download possible to make do and keep everything loading in the background and _hopefully_ when the user reaches further into the game our assets will be ready.  
To this purpose, PixiJS includes [Assets](https://pixijs.download/v7.4.2/docs/PIXI.Assets.html): A system for your resource management needs. A promise-based, background-loading, webworker-using, format-detecting, juggernaut of a system.

In this recipe, we are going to create one of our `Scene` to load all the files we declare in a manifest object and then I will teach you how to recover the files from the `Assets` cache.

## First of all, a primer on _Promises_.

> use `.then(..)` to be notified when that value is ready

```ts
somePromise.then((myValue) => {
    // This function will be called when "myValue" is ready to go!
});
```

A promise is something that _will eventually have a value_ for you to use but you can't be sure if that value is ready or not just yet, so you ask nicely to be notified when that happens.  
When a Promise has your value, it is said that it _resolves_ your value. For you to use the value you need to give a function to the `.then(...)` method.

> keep in mind that even resolved promises take a bit of time before executing your then, so take a look at this code...

```ts
console.log("before");
alreadyResolvedPromise((value)=>{
    console.log("inside the then with my value", value);
});
console.log("after")
```

> will give us: "before" -> "after" -> "inside the then with my value".

<aside class="info">Even if the promise was resolved long ago, adding a <code>.then(...)</code> will trigger almost immediately, this means you don't have to <i>see it</i> resolve, you can come after it has resolved and it will still give you a value.</aside>

### Awaiting Promises

> Take a look at this very naïve code...

```ts
// async functions always return a Promise of whatever they return
async function waitForPromise() : Promise<string>
{
    console.log("This starts excecuting");

    // As this next line has `await` in it, it will "freeze" the execution here until that magical promise resolves to a value.
    const magicalNumber = await someMagicalPromise;

    // When the promise is resolved, our code keeps executing
    console.log("Wow, that took a while!");

    // We return a string with the resolved value... but that took some time, so the entire function actually returns a promise.
    return "This will be finally resolved! The Magical number was " + magicalNumber.toString(); 
}

// We kinda need to call the function, right?
waitForPromise();
```

To keep code somewhat easier to read, you can use two keywords `async` and `await` to _freeze_ the code until a promise resolves.  
This doesn't really change how Promises work, this is only _syntax sugar_ that will be turned into the regular functions by the browser.  

<aside class="warning">
Some rules about <code>async</code> and <code>await</code>:
<ul>
<li><code>await</code> can only be used inside an <code>async</code> function.</li>
<li>a Constructor can never be <code>async</code></li>
<li>try to avoid mixing <code>.then(...)</code> and <code>await</code> inside an <code>async</code> function if you can. It can get confusing very fast</li>
</ul>
</aside>

## Using _Assets_

> Using `Assets.load(...)` with `.then(...)`

```ts
Assets.load("./clampy.png").then(clampyTexture => {
    const clampySprite = Sprite.from(clampyTexture);
});
// Be careful, `clampySprite` doesn't exist out here, only inside those brackets!
```

> Using `Assets.load(...)` with `async` `await`

```ts
async function loadClampy() : Promise<Sprite>
{
    const clampyTexture = Assets.load("./clampy.png");
    return Sprite.from(clampyTexture);
}
// now calling `loadClampy()` will yield a Clampy sprite promise!
```

Sometimes we just want to load one single file, quick and dirty. For that purpose we have `Assets.load(...)`.  
This will return a promise that will solve with the loaded and parsed asset ready to use! 

<aside class="info">Assets is a very smart piece of code and it will never download twice the same asset! Feel free to call <code>Assets.load(...)</code> every time you need an asset and it will give you the cached asset if it was already downloaded.</aside>

### .load(...) vs .get(...)

`Assets` has two methods to get your asset from inside it. The safer `Assets.load(...)` that will return a promise and the unsafe `Assets.get(...)` that will just return your asset... or `undefined`.  
If you are _absolutely_ sure that your asset has been loaded before and you don't want to deal with the promise that `Assets.load(...)` returns, you can use `Assets.get(...)` to get your asset directly from the cache, however if you were wrong and your asset wasn't loaded before you will get `undefined`.

## The files-to-download Manifest.

> Let's start with our _manifest_ object. I will call this file `assets.ts` {manifest's entries can have many shapes, this is the one I like the most.}

```ts
import type { ResolverManifest } from "pixi.js";

export const manifest:ResolverManifest = {
    bundles: [
        {
            name : "bundleName",
            assets:
            {
                "Clampy the clamp" : "./clampy.png",
                "another image" : "./monster.png",
            }
        },
        {
            name : "another bundle",
            assets:
            {
                "whistle" : "./whistle.mp3",
            }
        },
    ]
}
```

As javascript is unable to "scan" a directory and just load everything inside we need to add some sort of _manifest object_ that lists all the files that we need to download. Conveniently, `Assets` has a format to feed _Asset Bundles_ from a _manifest_.  
An _Asset Bundle_ is just that, a group of assets that make sense for our game to download together, for example, you make an _bundle_ for each screen of your game.  
Here we can see we are how we can declare (and export for outside use) an _Asset Bundle_ with the format that `Assets` wants it.  
Each bundle has a `name` field and an object of `assets` where the _key_ is the name for each asset and the _value_ is the URL for it. (by starting with `./` we mean "relative to our _index.html_").  

<aside class="info">Remember that Assets never downloads the same asset twice so don't be afraid to put the same asset in two different bundles.</aside>

## Using our _Manifest_

> This is just a snippet of how to initialize `Assets`. After this, we will see how to make a full loader `Scene` and background loading.

```ts
// remember the assets manifest we created before? You need to import it here
async function initializeLoader(): Promise<void> // This promise won't return any value, will just take time to finish.
{
    // Make sure you don't use Assets before this or you will get a warning and it won't work!
    await Assets.init({ manifest: manifest });

    // let's extract the bundle ids. This is a bit of js black magic
    const bundleIds =  manifest.bundles.map(bundle => bundle.name);

    // we download ALL our bundles and wait for them
    await Assets.loadBundle(bundleIds);

    // Code ends when all bundles are loaded!
}

// Remember to call it and remember it returns a promise
initializeLoader().then(() => {
    // ALL your assets are ready!
});
```

> This is enough to start downloading assets... but what about progress? and how can we tell when we are done?

`Assets` is the global static instance in charge of resolving, downloading and keeping track of all our assets so we never download twice. We initialize with `Assets.init(...)` feeding it our manifest and then we call `Assets.loadBundle(...)` with _all_ the names of our _asset bundles_.  

## Making it look pretty

> This is our full code for `LoaderScene.ts`

```ts
import { Container, Graphics, Assets } from "pixi.js";
import { manifest } from "../assets";

export class LoaderScene extends Container {

    // for making our loader graphics...
    private loaderBar: Container;
    private loaderBarBoder: Graphics;
    private loaderBarFill: Graphics;
    constructor(screenWidth: number, screenHeight: number) {
        super();

        // lets make a loader graphic:
        const loaderBarWidth = screenWidth * 0.8; // just an auxiliar variable
        // the fill of the bar.
        this.loaderBarFill = new Graphics();
        this.loaderBarFill.beginFill(0x008800, 1)
        this.loaderBarFill.drawRect(0, 0, loaderBarWidth, 50);
        this.loaderBarFill.endFill();
        this.loaderBarFill.scale.x = 0; // we draw the filled bar and with scale we set the %

        // The border of the bar.
        this.loaderBarBoder = new Graphics();
        this.loaderBarBoder.lineStyle(10, 0x0, 1);
        this.loaderBarBoder.drawRect(0, 0, loaderBarWidth, 50);

        // Now we keep the border and the fill in a container so we can move them together.
        this.loaderBar = new Container();
        this.loaderBar.addChild(this.loaderBarFill);
        this.loaderBar.addChild(this.loaderBarBoder);
        //Looks complex but this just centers the bar on screen.
        this.loaderBar.position.x = (screenWidth - this.loaderBar.width) / 2; 
        this.loaderBar.position.y = (screenHeight - this.loaderBar.height) / 2;
        this.addChild(this.loaderBar);

        // Start loading!
        this.initializeLoader().then(() => {
            // Remember that constructors can't be async, so we are forced to use .then(...) here!
            this.gameLoaded();
        })
    }

    private async initializeLoader(): Promise<void>
    {
        await Assets.init({ manifest: manifest });

        const bundleIds =  manifest.bundles.map(bundle => bundle.name);

        // The second parameter for `loadBundle` is a function that reports the download progress!
        await Assets.loadBundle(bundleIds, this.downloadProgress.bind(this));
    }

    private downloadProgress(progressRatio: number): void {
        // progressRatio goes from 0 to 1, so set it to scale
        this.loaderBarFill.scale.x = progressRatio;
    }

    private gameLoaded(): void {
        // Our game finished loading!

        // Let's remove our loading bar
        this.removeChild(this.loaderBar);

        // all your assets are ready! I would probably change to another scene
        // ...but you could build your entire game here if you want
        // (pls don't)
    }
}
```
> Keep in mind that if you have few assets (or a really fast internet connection), you might not see the progress bar at all!

I will not explain how the loading bar was made. If you need to refresh how to put stuff on screen check [that chapter again](#putting-stuff-on-screen).

You will see that the constructor calls an `async` function and is forced to use the `.then(...)` method, that is because constructors can never be `async`. 
You might have realized that we just downloaded _all_ our assets at the same time, at the very beginning of our game. This is has the benefit that once we leave that loading screen we will never need to download anything else ever again, however we forced our user to stay some time waiting and users nowadays don't like waiting.  
To remedy this we can take a different approach: we just load the bare minimum needed for the current screen and let the rest download in the background. We will see this in a [future recipe](#recipe-lazy-loading-assets)

<aside class="warning">You might have noted that I used only <code>Graphics</code> to make my loader progress bar. This is because if I were to use assets I would need a loader for the loader assets and that would mess with the space-time continuum. However, should you want your loader to have assets you can reference them directly by URL (<code>Sprite.from(urlHere)</code>) but make sure you call <code>Asssets.init(...)</code> before that, otherwise <code>Assets</code> will auto-initialize itself and your loader will break with a cryptic warning.</aside>


## How to easily use your loaded Sprites and Sounds?

Now that we downloaded and safely stored our assets in a cache... how do we use them?  
The two basic components we have seen so far are `Sprite` and `Sound` and we will use them in different ways.  

For _Sprites_, all our textures are stored in a [TextureCache somewhere in the PixiJS universe](https://pixijs.download/v7.4.2/docs/PIXI.utils.html#TextureCache) but all we need to know is that we can access that cache by doing `Sprite.from(...)` but instead of giving an URL we just give the name we gave our asset in the manifest file. In my example above I could do `Sprite.from("Clampy the clamp")`.  (If you ever need a `Texture` object, you can get it the same way with `Texture.from(...)` just remember that _Sprites_ go on screen and _Textures_ hide inside sprites.)

For _Sounds_, all our sounds are stored in what PixiJS Sound calls _sound library_. To access it we have to import it as `import { sound } from "@pixi/sound";`. That is `sound` with a **lowercase s**. From there you can play it by doing `sound.play("name we gave in the manifest");`.

<aside class="warning">
PixiJS Sound names can be confusing, while <code>Sound</code> with uppercase is the class for a single sound file, <code>sound</code> with lowercase is the sound library object.
</aside>

<aside class="info">
Remember you can always access your assets with <code>Assets.get("the name you gave your asset in the manifest");</code> and that will return the correct object (for example <code>Texture</code> or a <code>Sound</code>) as long as it was already loaded before. If you are not sure if the asset was loaded before, you should use <code>Assets.load(...);</code>
</aside>

## Advanced loading

Ok, we have seen how to load our basic _png_ and _mp3_ but what about other kinds of assets? What about _spritesheets_, _fonts_, _levels_, and more?  
This section will show some of the most common asset types and how to load and use them.

But before we start, a bit of how things will work behind the curtains: _Assets_, _Resolvers_, _Loader_, _Cache_ and _Detection_. 
You might never need to know more about these, but I have to mention them.  

- [Assets](https://pixijs.download/v7.4.2/docs/PIXI.Assets.html) is a static instance for `AssetsClass`. It's the entry point and coordinator of the rest of the system.
- [Resolver](https://pixijs.download/v7.4.2/docs/PIXI.Resolver.html) is a class that knows how to map assets to URLs. It also knows how to pick the best asset type for your system.
- [Loader](https://pixijs.download/v7.4.2/docs/PIXI.AssetLoader.html) is the bit that actually downloads your file. It does noting else than going to an URL and getting data from it.
- [Cache](https://pixijs.download/v7.4.2/docs/PIXI.Cache.html) stores all the related assets for a certain key.
- [Detection](https://pixijs.download/v7.4.2/docs/PIXI.FormatDetectionParser.html) is how you tell the rest of the system that you can totally handle that format of asset.

### Spritesheets

> Add this to one of your bundles in `assets.ts`

```ts
...
    name : "bundleName",
    assets:
    {
        ...
        "you probably wont use this name": "./yourSpritesheetUrl.json",
        // Don't add an entry for the .png file! Just make sure it exists next to your json file and it will work.
        ...
    }
...
```

> Then your textures from inside your spritesheet will exist in the cache! Ready to Sprite.from()

```ts
Sprite.from("Name from inside your spritesheet");
// or
Texture.from("Name from inside your spritesheet");
```

A spritesheet (also known as texture atlas) is a single image file with many assets inside next to a text file (in our case a `json` file) that explains how to slice that texture to extract all the assets. It is really good for performance and you should try to always use them.  
To create a spritesheet you can use paid programs like [TexturePacker](https://www.codeandweb.com/texturepacker) or free ones like [ShoeBox](https://renderhjs.net/shoebox/) or [FreeTexPacker](http://free-tex-packer.com/).  
If you have problems finding a _PixiJS_ compatible format in the packer of your choice, it might also be called _JSON Hash format_

PixiJS includes a spritesheet parser so all you need to do is provide the _url_ for the _json_ file. **You shouldn't add the URL for .png file but it must be next to the json file!**

### Fonts
> Add your fonts to a bundle in your `assets.ts`

```ts
...
    name : "bundleName",
    assets:
    {
        ...
        "you probably wont use this name but don't repeat names": "./fonts/open-sans.woff2",
        ...
    }
...
```

> Your fonts will be registered into the system and you can just ask for the Font Name. You can check the font name by opening your font file and checking the name. Also, remember you can make your text style with the [PixiJS Textstyle editor](https://pixijs.io/pixi-text-style).

```ts
const customStyleFont: TextStyle = new TextStyle({
    fontFamily: "Open Sans Condensed", // Your official font name
});
new Text('Ready to use!', customStyleFont); // Text supports unicode!
```

In PixiJS v7 fonts are now first class citizens!
To use a custom font your just need to add the font to your manifest, it can be in any format: `ttf`, `otf`, `woff` or `woff2`.

<aside class="info">
On PixiJS v6 you needed a plugin to load fonts but not anymore. Feel free to remove it if you have it.
</aside>


### Maps from level editors or custom txt, json, xml, etc.

> Just add your custom file to your `assets.ts`

```ts

...
    name : "bundleName",
    assets:
    {
        ...
        "my text": "./myTextFile.txt",
        "my json": "./myJsonFile.json",
        "my xml": "./myXMLFile.xml",
        ...
    }
...
```

> A good way to see what you got is to `console.log()`

```ts
// I can use `Assets.get(...)` because I am 100% sure that the assets are loaded!
// When in doubt use `Assets.load(...)`!
console.log(Assets.get("my text")); 
console.log(Assets.get("my json")); 
console.log(Assets.get("my xml")); 
```

The _Assets_ class will recognize simple text formats like `txt`, `json` or `xml` and will give out a somewhat usable object.

<aside class="info">
If a json file happens to be a spritesheet, the Loader will recognize it automatically and process it accordingly. No extra steps are needed!
</aside>

If you would like to parse a particular kind of file you will need to write your own _Assets plugin_. The best way is to look at the [Spritesheet](https://github.com/pixijs/pixijs/blob/v7.4.2/packages/spritesheet/src/spritesheetAsset.ts) or [BitmapFont](https://github.com/pixijs/pixijs/blob/v7.4.2/packages/text-bitmap/src/loadBitmapFont.ts) one and work from there.