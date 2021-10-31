# Getting Started

## Before we even start...
This tutorial won't teach you to code from scratch. I will assume you already know your way around code and object-oriented programming.  
Git and some shell (console) experience will be good but not mandatory. If you know any object-oriented programming you will be just fine.  

### This ain't your grandma's Javascript...
If you came here expecting to write javascript between two `<script>` tags in your .html file, oh boy I have bad news for you.  
Web development might have started with only a text editor and dumping code into a single .js file but in today's spooky fast world of javascript libraries, going commando ain't gonna cut it.  
This... guide? course? tutorial? will use Typescript and it will be designed to work with [_pixi-hotwire_](https://github.com/miltoncandelero/pixi-hotwire), a boilerplate to get you started fast without the hassle of how to create and maintain a standard web project with a bundler. (If you have a bit more experience in web development, feel free to tinker and try any bundler of your liking.)    

### pixi-hotwire?
In case you realize you have no idea what a bundler is or why you need one, worry not! I've created a base template project called [_pixi-hotwire_](https://github.com/miltoncandelero/pixi-hotwire). It is the simplest boilerplate I could build. It goes from nothing to watching your code running with the least amount of configurations.  
It is a _just-add-npm_ solution that sets you up with _typescript_, _pixi.js_ and _webpack_. It also will make a final build folder for you instead of you having to pluck manually files out of your project folder to get the _uploadeable_ result of your hard work.

### Typescript?
I'll be honest with you: I hate Javascript. I hate the meme of truthy and falsy values. I hate the implicit type coercion. I hate the dynamic context binding. I hate a lot of javascript... But I must endure, adapt, overcome.  
Enter Typescript: Typescript will force your hand to keep your code as strictly typed as possible, and if you need to use type coercion and really _javascripty_ code, you must consciously choose to make the data type as `any`.  
As a nice bonus, you get _intellisense_ on Visual Studio Code (which is Microsoft's fancy name for code auto-complete).  

### I will never scream (`PIXI.`)
I feel that doing `import * as PIXI from "pixi.js"` so that I can go `PIXI.Sprite`, `PIXI.Container`, `PIXI.Loader`, etc, is silly and makes me feel clumsy. I will use named imports: `import { Sprite, Container, Loader } from "pixi.js"` and that allows me to just use the class name without adding `PIXI` everywhere.  
I could say it is better for three-shaking (the step where the bundler doesn't add to your code stuff that you never use) or that I fight the smurf naming convention... but at the end of the day I just like named imports better. Sorry, not sorry.  

## Enough talk, have at you!

> Be patient. When the time is right, the code will be shown in this column.

Let's start by making sure you have all the tools and materials.  

You will need:

* [NodeJS](https://nodejs.org/) (it comes with `npm`).
* [pixi-hotwire](https://github.com/miltoncandelero/pixi-hotwire) boilerplate.
* A text editor (consider [Visual Studio Code](https://code.visualstudio.com/) but any would do)
* A web browser from this era. (Sorry internet explorer, you are out!)


A quick overview of what you will find inside _pixi-hotwire_:

* `src` folder: Contains the typescript code for your project.
* `static` folder: Contains all the assets (non-code) for your project.
* `package.json` file: Contains a list of all the libraries and tools you need along with some script code to run and build your project. The heart and soul of any `npm` project.
* `dist` folder: Won't be there yet. Here you will find the ready-to-upload build when you want to share your game.

Once you have cloned or downloaded _pixi-hotwire_ you will need to grab all the dependencies (stored inside the `package.json` file), to do so you will need to use a shell (console), navigate to the project folder and use the command `npm install` to read all the dependencies and download them into the `node_modules` folder.  

When the progress finishes you now have access to new stuff that begins with `npm`!

* `npm run start` will convert your typescript into javascript as you write it and let you test the game! 
  * "as you write" means you just need to save the file you are working in order to see the changes, no need to run it again!
* `npm run build` will create a package for you to upload to your webserver. Find it inside the folder `dist` (short for distributable)
  * Don't try to "just double click" the index.html file there. It may work, it may not. This is meant to upload into a webserver or service. When your game is ready you can try uploading to [itch.io](https://itch.io/).

Run `npm run start` and open your web browser on the website `http://localhost:1234`.  
`localhost` means "your own computer" and `1234` is the port where this web server is running. Nobody else will see your game running on their `localhost`. You will need to export it and upload it somewhere.  
Try hitting F12 and finding the Javascript Console. That will be your best friend when needing to know why something is or isn't working. That is where `console.log()` will write and where you will see any errors that you might have made.

## Finally some code!

> Now, for the actual code inside the `index.ts` file

```ts
import { Application, Sprite } from 'pixi.js'

const app = new Application({
	view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
	resolution: window.devicePixelRatio || 1,
	autoDensity: true,
	backgroundColor: 0x6495ed,
	width: 640,
	height: 480
});

const clampy: Sprite = Sprite.from("clampy.png");

clampy.anchor.set(0.5);

clampy.x = app.screen.width / 2;
clampy.y = app.screen.height / 2;

app.stage.addChild(clampy);
```

First, we see the `import` statement. As previously stated, I prefer named imports to just importing everything under a big all-caps `PIXI` object.  

After that, we create an `app` instance of PixiJS `Application`. This object is a quick way to set up a renderer and a stage to drop stuff on screen and be ready to render. You can see that I call for an HTML element by id `pixi-canvas`, this element can be found on the `index.html` file. (If you know a bit of web development, that file will reference directly the typescript file, which is usually illegal. This works because that file it's used by `parcel` as the entry point. The final exported index.html file won't have any reference to any typescript file.)  
As a parameter for the `Application` object, we give it some options. You can see and explore the full list in [PixiJS official docs](https://pixijs.download/dev/docs/PIXI.Application.html).  

Then, I create a `Sprite` with the `Sprite.from(...)` method, this is a super powerful shortcut that can take as a parameter one of many things, among which we can find:

* A `Texture` object.
* The name of a `Texture` object you loaded previously. (using a `Loader`, we will see this when we get to [loading assets](#recipe-preloading-assets))
* The URL of an image file

Can you guess what we used here?  
Well, I hope you guessed option three, "the URL of an image file" because that is the correct option.  
By saying `"clampy.png"` it means "find a file called clampy.png in the root of my asset folder". You will see that our asset folder is called `static` and if you check in there, indeed there it is, `clampy.png`!  

Finally, I set the position of Clampy and add it to the screen by doing an `addChild()` to the `app.stage`.  
You will learn soon enough what the stage and why is it called `addChild()` but for now it should suffice to know that `app.stage` is the name for "the entire screen" and everything that you want to show needs to be appended by using `addChild()`.

### Homework!
Oh boy, you thought you could get away?  
Try these on for size!

* Try moving the `clampy.png` into a folder (keeping it inside `static`!)
  * Now, try referencing the new location to make Clampy appear again
* Try using an image from the internet, say from [imgur](https://imgur.com/)
  * Make sure your file ends with `.png` or `.jpg`, which means it is the actual image file.
* Try an image from a random website. (it may work, it may not, don't panic)
  * Did you get a `blocked by CORS` error on the javascript console? [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is a safety measure to prevent a webpage from calling other webpages without your permission, this is why we have our own images on the `static` folder!.