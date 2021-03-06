# live-p5

This extension was based on the good work from [pixelkind](https://github.com/pixelkind/p5canvas).

It provides a live preview panel of your P5 code.

It enables you to change variable values without reloading the P5 rendering, see the gif below:

![In action](image.gif)

## How to use it

* Open your javascript p5 code with a `draw` function
* Type **"live p5"** on the command palette and press enter
* When editing literal values, the preview is updated automatically
* When saving the document, the preview reloads

## Caveats

### When **not** reloading is not a good thing

The extension tries its best to only reload P5 when necessary; it does this by analysing if a code change only affected literal values (numbers, booleans, and strings).

This is a problem when changing literals that are not used in the `draw` loop. For instance, if you change a literal that affects how the `setup` function works, P5 will only be reloaded when you save your document.

### Where do my `console.log`s and runtime errors go?

Because of how the preview panel works in vscode, prints and runtime errors get printed to the developer console.  

The original extension by pixelkind does a workaround for print statements, but there's no way around the runtime errors.  
Because of this I decided to remove the workaround completely and now it is necessary to have the developer console open when working with this extension.

Another caveat is that the preview panel does not implement `console.log` correctly, as it doesn't support multiple arguments. So sadly you must concatecate strings yourself, for example:

```javascript
console.log(1, 2, 3); // prints [Embedded page] 1
console.log("1, 2, 3"); // prints [Embedded page] 1, 2, 3
```

## How does it work?

It is not very pretty.

Using the excellent [recast](https://github.com/benjamn/recast) library, the extension transforms the code you typed into something else:

```javascript
function draw() {
  console.log(1);
}
```

Becomes:

```javascript
const __AllVars = {
  aHash: 1
};

function draw() {
  console.log(__AllVars['aHash']);
}
```

When editing your code, if the `1` literal is changed to `11`, the extension sends an updated `__AllVars` hash to the preview panel using websockets, and updates the hash in memory, thus not reloading the panel but affecting what gets rendered.

If there are any changes to the code's structure, the panel is reloaded automatically.
