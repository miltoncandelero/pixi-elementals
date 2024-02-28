# Splitting code
_it's not me, it's you_

We have been dumping all our code directly into  `index.ts` and this is all fine and dandy for quick tests and learning stuff but if we are going to move forward we need to learn how to split up our codebase into different files.  

## A quick preview of Scenes

> This is my `Scene.ts` file

```ts
import { Container, Sprite } from "pixi.js";

export class Scene extends Container {
    private readonly screenWidth: number;
    private readonly screenHeight: number;

    // We promoted clampy to a member of the class
    private clampy: Sprite;
    constructor(screenWidth: number, screenHeight: number) {
        super(); // Mandatory! This calls the superclass constructor.

        // see how members of the class need `this.`?
        this.screenWidth = screenWidth;
        this.screenHeight = screenHeight;

        // Now clampy is a class member, we will be able to use it in another methods!
        this.clampy = Sprite.from("clampy.png");

        this.clampy.anchor.set(0.5);
        this.clampy.x = this.screenWidth / 2;
        this.clampy.y = this.screenHeight / 2;
        this.addChild(this.clampy);
    }
}
```

In a future chapter I will [provide an example of my scene management system](#recipe-scene-manager) but for now, let's start by making a class that inherits from Container and that will be the lifecycle of our examples and demos. This will allow us to have methods that come from an object instead of being global functions.  
Start by creating a new `.ts` file and create a class that extends `Container`. In my case, I created `Scene.ts` and created the `Scene` class. (I like to name my files with the same name as my class and to keep my classes starting with an uppercase).  
See the `export` keyword? In this wacky modular world, other files can only see what you export.  
And in the constructor of this new class, let's dump the code of our getting started.

<aside class="warning">
See how I passed the screen size as a parameter to the constructor and stored it into a variable? That is to avoid "asking up" in my mental map of classes. A class should only know classes below itself. If you are going to need globally available stuff, try to limit it to avoid spaghetti code.
</aside>

> And this is how my `index.ts` looks now

```ts
import { Application } from 'pixi.js'
import { Scene } from './scenes/Scene'; // This is the import statement

const app = new Application<HTMLCanvasElement>({
	view: document.getElementById("pixi-canvas") as HTMLCanvasElement,
	resolution: window.devicePixelRatio || 1,
    autoDensity: true,
	backgroundColor: 0x6495ed,
	width: 640,
	height: 480
});

// pass in the screen size to avoid "asking up"
const sceny: Scene = new Scene(app.screen.width, app.screen.height);

app.stage.addChild(sceny)
```

Finally, let's go back to the `index.ts` file and create and add to the screen a new `Scene` instance!  
First, we will need to import it, Visual Studio Code usually can suggest you the import path, otherwise you just `import` it with curly braces and the relative or absolute path of the file.

<aside class="notice">
The curly braces in the import are there because a single file can export many classes or functions. However, unless you are exporting a class and helper functions or structures, I would strongly advise against having more than one class per file.
</aside>

From now on, the examples will show you "scenes" instead of raw code to plaster in your `index.ts`. You have been warned.