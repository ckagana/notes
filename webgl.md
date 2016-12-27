# WebGL

The WebGL technology allows the developer to use hardware-accelerated 3D graphics in the browser, not unlike what the OpenGL library does for environments such as C and C++.

It's embedded into the [latest versions of all modern browsers](http://caniuse.com/#feat=webgl), and as such it can be used straight away, without any additional software being required to be installed. Much like the DOM technology, WebGL exposes an API that enhances the capabilities of the JavaScript language on the client.

A WebGL program (not unlike its OpenGL relative) is based on the concept of the *scene*, which is the set of graphical objects that have to be drawn on the screen. Since we are in an HTML5 environment, it's not possible to draw freely everywhere on the screen. Instead, we are confined inside the boundaries of the `<canvas>` element. This means that the entire scene we are going to draw using WebGL will be placed inside one `<canvas>` element. More precisely, using the WebGL API we can select the canvas we want to draw into, using standard DOM selection tools like the DOM API, or jQuery. This means that we can add more than one canvas to the page, and then write some code targeting one canvas, in addition to some other code targeting others.

A basic example of an HTML5 canvas at work can be this:

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>OpenGL canvas example</title>
        <style type="text/css">
            canvas { border: 2px dotted blue; }
        </style>
    </head>
    <body>
        <canvas id="canvas-element-id" width="800" height="600">
            Your browser does not support HTML5
        </canvas>
    </body>
</html>
</html>
```

As we can see from this example, the `canvas` element accepts some text to be written in, that will be printed to the page in case this kind of element is not supported by the browser. In this example, the CSS snippet is there only to allow us to actually see the canvas' region on the page, since it has a white background color by default. The canvas element comes with two attributes, `width` and `height`, useful to define the size of the canvas. Were these attributes not defined, the default size of the canvas would be of 300 x 150 pixels.

To access the WebGL API we need to use the `getContext()` method, invoked on the canvas DOM object we want to draw in. This method returns a JavaScript object known as the WebGL *context*, which encapsulates all WebGL features. This means that we can get a different context for every canvas present in the page.

First of all, we need to ensure that WebGL is actually supported in the current browser:

```javascript
jQuery(function($) {
    var $canvas = $('#canvas-element-id');
    if ($canvas.length === 0) {
        throw 'Canvas element not found!';
    }

    var names = ['webgl', 'experimental-webgl', 'webkit-3d', 'moz-webgl'];
    var context;
    var canvas = $canvas[0];
    for (i in names) {
        try {
            context = canvas.getContext(names[i]);
        } catch (e) {}
        if (context) { break; }
    }

    if (context === null) {
        throw 'WebGL not available';
    }
});
```

The `getContext()` method takes the context name as an argument. Unfortunately, this name varies from vendor to vendor, so we have to try them all, and take the first that works. Actually, the preferred context names are `webgl` and `experimental-webgl`. `webkit-3d` seems to have been recently (as of 2014) [removed from Chrome](https://code.google.com/p/chromium/issues/detail?id=357296). The same is true for [Mozilla's `moz-webgl`](https://bugzilla.mozilla.org/show_bug.cgi?id=913597).

All features of a WebGL context depend on the context's state, which is the set of values of all its attributes at any given moment. Working with a WebGL context is often a matter of reading or changing its state.

One of the most simple example of this, is the so-called "clear color" attribute, which controls the canvas' background color. To access any attribute of a context, we should use its `getParameter()` method, passing the attribute code to it. The attribute code for the "clear color" is stored inside the context's property `COLOR_CLEAR_VALUE`. So, to get the current clear color:

```javascript
var color = context.getParameter(context.COLOR_CLEAR_VALUE);
```

The "clear color" attribute, as any other context attribute, cannot be directly overwritten, as it's not a regular property of the context object, and its implementation is encapsulated by it. To change the value of the clear color attribute, we should use the `¢learColor()` method, as:

```javascript
context.clearColor(0.3, 0.7, 0.2, 1.0);
```

Of course the argument of this method are the RGBA values for the color we want to clear the background with. Now the clear color of our context is appropriately updated, but the canvas element is still unaffected. To make the canvas reflect this state change we should use the `clear()` method, which is used to reset a specific buffer:

```javascript
context.clear(context.COLOR_BUFFER_BIT);
```

Here, the `COLOR_BUFFER_BIT` constant identifies another context attribute, the "color buffer", which stores the colors of the scene. The `clear()` method automatically reads the `COLOR_CLEAR_VALUE` attribute, and resets the color buffer to the color written therein. The color buffer contains the colors used by the canvas element, so after changing it, the canvas is directly updated.

The steps previously described, define a common pattern used when working with WebGL: everything is done reading and updating the context state, and every feature depends on some state attribute.
