# Sounding Good
_You and the marvelous world of sounds_

To play sounds and music in our games we are going to use the [PixiJS solution for sound](https://github.com/pixijs/sound).  
Just like its rendering counterpart, PixiJS Sound hides away an implementation meant to work on every browser leveraging the WebAudio API giving us a simple-to-use interface with a lot of power under the hood.  
If you want a more in-depth demo you can check the [PixiJS Demo website](https://pixijs.io/sound/examples/index.html).

## Playing a sound

```ts
import { Container } from "pixi.js";
import { Sound } from "@pixi/sound";

export class Scene extends Container {
    constructor(screenWidth: number, screenHeight: number) {
        super();

        // Just like everything else, `from()` and then we are good to go
        const whistly = Sound.from("whistle.mp3");
        whistly.volume = 0.5;
        whistly.play();

    }
}
```

If you reached this point making `Sprites` you will see the `Sound.from()` syntax and feel at home.  
You just create a sound object directly from a sound file URL and then you can tweak it and play it!  
Some helpful things inside a sound are:

* `.play()`, `.stop()` and `.isPlaying`
* `.pause()`, `.resume()` and `.paused`
* `.volume` and `.muted`
* `.loop`
* `.filters` (this is a bit more advanced but you can achieve some really cool effects!)

<aside class="info">You think you did everything correctly but your sounds aren't playing? Tried clicking inside your game? Now they fired all at once and are working?  
This is due to a policy that started on Apple's Safari and then extended into all browsers where you can't play a sound without user interaction. To circumvent this, PixiJS Sound creates an invisible giant button to <b>unlock</b> your audio. After the user clicks once we can play all the sounds we want.  
<a href="https://developers.google.com/web/updates/2017/09/autoplay-policy-changes">For more information, Chrome's explanation is quite good.</a></aside>