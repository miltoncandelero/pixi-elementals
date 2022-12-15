# Recipe: Preloading assets v6
_Superseded by the `Assets` class in PixiJS v7_

So far we have seen how to create images and sounds by downloading the asset behind them just as we need to show it to the user. While this is good enough if we want a quick way to make a proof of concept or prototype, it won't be good enough for a project release.  
The elegant way of doing it is downloading all the assets you are going to need beforehand and storing them in some sort of cache. To this purpose, PixiJS includes [Loader](https://pixijs.download/v6.5.8/docs/PIXI.Loader.html): An extensible class to allow the download and caching of any file you might need for your game.  

In this recipe, we are going to create one of our `Scene` to load all the files we declare in a manifest object and then I will teach you how to recover the files from the `Loader` cache.

## The file to download manifest.

> Let's start with our _manifest_ object. I will call this file `assets.ts`

```ts
export const assets = [
    { name: "Clampy the clamp", url: "./clampy.png" },
    { name: "another image", url: "./monster.png" },
    { name: "whistle", url: "./whistle.mp3" },
]
```

As javascript is unable to "scan" a directory and just load everything inside we need to add some sort of _manifest object_ that lists all the files that we need to download.
Here we can see we are how we can declare (and export for outside use) an array of objects that will be used by the `Loader`.  
The `name` field is going to be our _key_ to retrieve the downloaded object and the `url` field must point to where our asset is going to be located (by starting with `./` we mean "relative to our _index.html_").  

## How to use the _Loader_

> This is just a snippet of how to use `Loader`. After this, we will see how to make a full loader `Scene`

```ts
// remember the assets manifest we created before? You need to import it here
Loader.shared.add(assets);

// this will start the load of the files
Loader.shared.load();

// In the future, when the download finishes you will find your entire asset like this
Loader.shared.resources["the name you gave your asset in the manifest"]; 
// You will probably want `.data` or `.texture` or `.sound` of this object 
// however for Pixi objects there is are better ways of creating them...
```

> This is enough to start downloading assets... but what about progress? and how can we tell when we are done?

Like the `Ticker` class we saw before (and many other PixiJS classes), `Loader` has a _shared_ instance we can access globally and we are going to use that to load our assets and keep them in _cache_ so we can reference them without downloading them each time.  
The `add(...)` method can take many shapes but we are feeding it our resource manifest we stored in `assets.ts` and then begin the download by calling the `load()` method.

## Making it look pretty

> This is our full code for `LoaderScene.ts`

```ts
import { Container, Graphics, Loader } from "pixi.js";
import { assets } from "../assets";

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

        // Now the actual asset loader:

        // we add the asset manifest
        Loader.shared.add(assets);

        // connect the events
        Loader.shared.onProgress.add(this.downloadProgress, this);
        Loader.shared.onComplete.once(this.gameLoaded, this);

        // Start loading!
        Loader.shared.load();
    }

    private downloadProgress(loader: Loader): void {
        // Progress goes from 0 to 100 but we are going to use 0 to 1 to set it to scale
        const progressRatio = loader.progress / 100;
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

Those events don't look exactly like the other events we saw in the [Interaction section](#getting-interactive) and that is because _Loader_ uses their own kind of events called _signals_ or _minisignals_ however they work quite similarly to regular events, the big difference is that each _signal_ is only good for one kind of event and that is why we don't have a string explaining what kind of event we need and instead we have two objects `onProgress` and `onComplete`.

<aside class="info">You might have noted that I used only <code>Graphics</code> to make my loader progress bar. This is because if I were to use assets I would need a loader for the loader assets and that would mess with the space-time continuum. However, should you want your loader to have assets you can reference them directly by URL (<code>Sprite.from(urlHere)</code>) or make your loader... loader and indeed destroy space-time itself.</aside>

## How to easily use your loaded Sprites and Sounds?

Now that we downloaded and safely stored our assets in a cache... how do we use them?  
The two basic components we have seen so far are `Sprite` and `Sound` and we will use them in different ways.  

For _Sprites_, all our textures are stored in a [TextureCache somewhere in the PixiJS universe](https://pixijs.download/dev/docs/PIXI.utils.html#TextureCache) but all we need to know is that we can access that cache by doing `Sprite.from(...)` but instead of giving an URL we just give the name we gave our asset in the manifest file. In my example above I could do `Sprite.from("Clampy the clamp")`.  (If you ever need a `Texture` object, you can get it the same way with `Texture.from(...)` just remember that _Sprites_ go on screen and _Textures_ hide inside sprites.)

For _Sounds_, all our sounds are stored in what PixiJS Sound calls _sound library_. To access it we have to import it as `import { sound } from "@pixi/sound";`. That is `sound` with a **lowercase s**. From there you can play it by doing `sound.play("name we gave in the manifest");`.

<aside class="warning">
PixiJS Sound names can be confusing, while <code>Sound</code> with uppercase is the class for a single sound file, <code>sound</code> with lowercase is the sound library object.
</aside>

<aside class="info">
Remember you can always access your assets with <code>Loader.shared.resources["the name you gave your asset in the manifest"];</code> and then use <code>.texture</code> for making sprites or <code>.sound</code> for playing sounds directly.
</aside>

## Advanced loading

Ok, we have seen how to load our basic _png_ and _mp3_ but what about other kinds of assets? What about _spritesheets_, _fonts_, _levels_, and more?  
This section will show some of the most common asset types and how to load and use them.

But before we start, a bit of how things will work behind the curtains: _Loader plugins_.  
The PixiJS Loader extends [Resource Loader by Chad Engler](https://github.com/englercj/resource-loader) and includes some plugins for downloading and parsing images, and spritesheets however for other types we will need custom plugins (e.g. for WebFonts) or the plugin will come bundled with a library (e.g. PixiSound includes a loader plugin)

### Spritesheets

> Add this to your `assets.ts` array...

```ts
{ name: "you wont use this name", url: "./yourSpritesheetUrl.json" }
// Don't add an entry for the .png file! Just make sure it exists next to your json file and it will work.
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
> Install PixiJS Webfont Loader by running `npm i pixi-webfont-loader` and then proceed to run this code **before using your loader**

```ts
// Add before using Loader!!!
Loader.registerPlugin(WebfontLoaderPlugin);
// Now you can start using loader like Loader.shared.add(assets);
```

> Add your .css fonts to your `assets.ts` array

```ts
{ name: "you wont use this name", url: "./fonts/stylesheet.css" }
```

> Your fonts will be registered into the system and you can just ask for the Font Name. You can check the font name by opening your `.css` with a text editor and checking the `font-family` inside. Also, remember you can make your text style with the [PixiJS Textstyle editor](https://pixijs.io/pixi-text-style).

```ts
const customStyleFont: TextStyle = new TextStyle({
    fontFamily: "Open Sans Condensed", // name from inside the .css file
});
new Text('Ready to use!', customStyleFont); // Text supports unicode!
```

When I showed you text examples I used system fonts on purpose, but what if you want to use your custom font? or a google font?  
In the web world, there are many font formats: `ttf`, `otf`, `woff`, `woff2`; but how do we make sure we have the right font format for each browser? Simple, WE HAVE THEM ALL!  
An easy way is to get the font file you want to use and convert it using a service like [Transfonter](https://transfonter.org/). That will give you a `.css` file and all the formats required.  
Another way is to go to [Google Fonts](https://fonts.google.com/) and download a Webfont from there.  

To load our font's `css` file we are going to need a _Loader plugin_: [PixiJS Webfont Loader](https://github.com/miltoncandelero/pixi-webfont-loader).  
You can install the plugin by running `npm i pixi-webfont-loader` in a console in your project and then inside your code you must register the plugin.

<aside class="info">
If you don't want to trouble yourself converting your original font file (<code>ttf</code>, <code>otf</code>, <code>woff</code>, or <code>woff2</code>) font into a <code>.css</code> Webfont you can still use it by loading it directly as long as you are using a version of PixiJS Webfont Loader equal or greater than 1.0.0.
</aside>


### Maps from level editors or custom txt, json, xml, etc.

> Just add your custom file to your `assets.ts`

```ts
{ name: "my text", url: "./myTextFile.txt" },
{ name: "my json", url: "./myJsonFile.json" },
{ name: "my xml", url: "./myXMLFile.xml" },
```

> A good way to see what you got is to `console.log()` the `data`

```ts
console.log(Loader.shared.resources["my text"].data); 
console.log(Loader.shared.resources["my json"].data); 
console.log(Loader.shared.resources["my xml"].data); 
```

The _Loader_ class will recognize simple text formats like `txt`, `json` or `xml` and will give out a somewhat usable object inside the `data` object of the resource.

<aside class="info">
If a json file happens to be a spritesheet, the Loader will recognize it automatically and process it accordingly. No extra steps are needed!
</aside>

If you would like to parse a particular kind of file you will need to write your own _Loader plugin_. I might write a tutorial for that in the future but for now, you can read [this one by Matt Karl](https://medium.com/@bigtimebuddy/new-pixijs-v5-plugins-75a7d86afb6).