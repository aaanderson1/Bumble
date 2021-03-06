# Bumble
A small JavaScript game framwork with a focus on coroutines.
<p align="center">
  <img src="https://raw.githubusercontent.com/jbluepolarbear/Bumble/master/bumble.png"/>
</p>

## Road Map
* Collision Utilities


## Sample Usage
```javascript
import { Bumble, BumbleUtility, BumbleVector, BumbleColor, BumbleKeyCodes, BumbleTransformation } from './bumble.js';
// create bumble instance
const bumble = new Bumble('sample', 720, 480, 'black', 60);
// can get and set gamestate that is saved in localStorage
const gamestate = bumble.gameState;
let timeRun = gamestate.getState('times run');
if (timeRun === null) {
    timeRun = 1;
}
gamestate.setState('times run', timeRun + 1);

bumble.preloader.loadAll([
    new BumbleResource('bumble', 'img/bumble.png', 'image'),
    new BumbleResource('laser', 'audio/laser.mp3', 'audio'),
    new BumbleResource('data', 'data/data.json', 'data')
]);

// all loading and game logic is run in coroutines
// coroutines support yielding naked yields, genertors, promises, and arrays of promises
bumble.runCoroutine(function *() {
    // load an image by url
    const image = bumble.getImage('bumble');
    const imageTransform = new BumbleTransformation(image.width, image.height);
    imageTransform.anchor = new BumbleVector(0.5, 0.5);
    imageTransform.position = new BumbleVector(image.width / 2, image.height / 2);

    // create a shape
    const shape = bumble.getShape([
        new BumbleVector(64, 0),
        new BumbleVector(128, 128),
        new BumbleVector(0, 128),
        new BumbleVector(64, 0)
    ], BumbleColor.fromRGB(0, 0, 255));
    const shapeTransform = new BumbleTransformation(shape.width, shape.height);
    shapeTransform.anchor = shape.getAnchorToCenterPoint();
    shapeTransform.position = new BumbleVector(shape.width / 2, shape.height / 2);
    shape.position = new BumbleVector(shape.width, shape.height);

    // coroutine to handle snaping the shape to the mouse on right/main click
    bumble.runCoroutine(function *() {
        while (true) {
            if (bumble.mouse.mouseState.buttonState[0]) {
                shapeTransform.position = bumble.mouse.mouseState.position.copy();
            }
            yield;
        }
    });

    const audio = bumble.getAudio('laser');
    bumble.runCoroutine(function *() {
        while (true) {
            if (bumble.keys.isDown(BumbleKeyCodes.SPACE)) {
                audio.play();
                yield BumbleUtility.wait(0.5);
            }
            yield;
        }
    });

    const data = bumble.getData('data');

    let angle = 0;
    // check keyboard input and left/alt mouse click
    while (!bumble.keys.isDown(BumbleKeyCodes.R) && !bumble.mouse.mouseState.buttonState[2]) {
        // clear canvas screen
        bumble.clearScreen();
        
        bumble.applyTransformation(imageTransform.build());
        image.draw();
        bumble.applyTransformation(shapeTransform.build());
        shape.draw();
        angle += 0.01;
        imageTransform.rotation = angle;
        shapeTransform.rotation = -angle;
        yield;
    }
});
```
