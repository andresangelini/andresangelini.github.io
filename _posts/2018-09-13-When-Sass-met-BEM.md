### When Sass met BEM

In the last post we created all the necessary tools for writing our CSS more efficiently. Now, we are ready to actually build the boards in the `_boards.scss` partial. We'll do this with the help of yet another tool: [BEM].

To be more precise, [BEM] is not actually a tool you can install, but a methodology through which we can write both our HTML and CSS more concisely by stablishing a clear structure of **Block**, **Element** and **Modifier**.

According to this new structure, we will define the **board** as a **block** since it has its own significance and can live independently from anything else. Then, we will handle the two types of boards as two different **modifiers**.

Let's open the `_boards.scss` partial and write the following:

```scss
// STYLING

.board {
  background-repeat: no-repeat;
  box-sizing: border-box;
  position: absolute;
}
```

We will using a `div` for every board we build so we definetely don't wan't the background to be repeated. Setting `background-repeat` to `no-repeat` will take care of that. We also want the padding and border in the `div` total height and width so we set `box-sizing` to `border-box`. Last but not least, we need the `position` to be `absolute` so we can use it as reference for the position of inner elements.

All these properties are what really defines a **board**. Everything else will be added as a modifier, so let's get started with the **bulletin** type by adding the following inside the `.board` rule:

```scss
// STYLING

.board {
  background-repeat: no-repeat;
  box-sizing: border-box;
  position: absolute;

  &--type-bulletin {
    /* Use SVG fragment identifiers to select specific parts of the image. */
    background-image: url("#{$path-to-board}#l-top-chains"),
                      url("#{$path-to-board}#l-plaque"),
                      url("#{$path-to-board}#l-bulletin-corners"),
                      url("#{$path-to-board}#l-bulletin-horizontal-sides"),
                      url("#{$path-to-board}#l-vertical-sides"),
                      url("#{$path-to-board}#l-bulletin-holes"),
                      url("#{$path-to-board}#l-horizontal-sides-depth"),
                      url("#{$path-to-board}#l-bulletin-corners-depth"),
                      url("#{$path-to-board}#l-horizontal-sides-shadow"),
                      url("#{$path-to-board}#l-vertical-sides-shadow"),
                      url("#{$path-to-board}#l-bulletin-corners-shadow"),
                      url("#{$path-to-board}#l-bulletin-holes-shadow"),
                      url("#{$path-to-board}#l-bulletin-planks");

    background-position: center #{-$bulletin-chains-position-top},        // #l-top-chains
                         center bottom,                                   // #l-plaque
                         center bottom,                                   // #l-bulletin-corners (diffuse and specular)
                         center bottom,                                   // #l-bulletin-horizontal-sides (diffuse and specular)
                         center $bulletin-vertical-position-top,          // #l-vertical-sides
                         center bottom,                                   // #l-bulletin-holes
                         center bottom,                                   // #l-horizontal-sides-depth
                         center bottom,                                   // #l-bulletin-corners-depth
                         center bottom,                                   // #l-horizontal-sides-shadow
                         center $bulletin-vertical-position-top,          // #l-vertical-sides-shadow
                         center bottom,                                   // #l-bulletin-corners-shadow
                         center bottom,                                   // #l-bulletin-holes-shadow
                         center bottom;                                   // #l-bulletin-planks

    background-size: $bulletin-holes-width $bulletin-chains-height,       // #l-top-chains
                     35% $bulletin-board-height,                          // #l-plaque
                     100% $bulletin-board-height,                         // #l-bulletin-corners (diffuse and specular)
                     $bulletin-horizontal-width $bulletin-board-height,   // #l-bulletin-horizontal-sides (diffuse and specular)
                     100% $bulletin-vertical-height,                      // #l-vertical-sides
                     $bulletin-holes-width $bulletin-board-height,        // #l-bulletin-holes
                     $bulletin-horizontal-width $bulletin-board-height,   // #l-horizontal-sides-depth
                     100% $bulletin-board-height,                         // #l-bulletin-corners-depth
                     $bulletin-horizontal-width $bulletin-board-height,   // #l-horizontal-sides-shadow
                     100% $bulletin-vertical-height,                      // #l-vertical-sides-shadow
                     100% $bulletin-board-height,                         // #l-bulletin-corners-shadow
                     $bulletin-holes-shadow-width $bulletin-board-height, // #l-bulletin-holes-shadow
                     100% $bulletin-board-height;                         // #l-bulletin-planks
  }
}
```

With the help of the almighty [Sass Ampersand] we nested our new class by prepending the `&` to `--type-bulletin` modifier to make reading our rules easier. Even though in SCSS you see it like this:

```scss
.board {
  &--type-bulletin {
    // ...
  }
}
```

The final output in CSS will still be:

```scss
.board {
  // ...
}

.board--type-bulletin {
  // ...
}
```

Besides that, we did basically three things here: set the background images, positioned them and set their respective sizes. But if you take a closer look, you will now appreciate how much work we managed to take off from our shoulders by creating those functions and variables earlier.

That interpolated `$path-to-board` variable not only saves us a lot of space but also lets us change the path in one single place. Same with `$bulletin-vertical-position-top`, `$bulletin-board-height`, etc.

Now let's do the same thing with the **sign** type by creating a new modifier also inside the `.board` rule.

```scss
&--type-sign {
    /* Use SVG fragment identifiers to select specific parts of the image. */
    background-image: url("#{$path-to-board}#l-top-chains"),
                      url("#{$path-to-board}#l-bottom-chains"),
                      url("#{$path-to-board}#l-sign-corners"),
                      url("#{$path-to-board}#l-sign-horizontal-sides"),
                      url("#{$path-to-board}#l-vertical-sides"),
                      url("#{$path-to-board}#l-sign-holes"),
                      url("#{$path-to-board}#l-horizontal-sides-depth"),
                      url("#{$path-to-board}#l-sign-corners-depth"),
                      url("#{$path-to-board}#l-horizontal-sides-shadow"),
                      url("#{$path-to-board}#l-vertical-sides-shadow"),
                      url("#{$path-to-board}#l-sign-corners-shadow"),
                      url("#{$path-to-board}#l-sign-holes-shadow"),
                      url("#{$path-to-board}#l-sign-planks");

    background-position: center #{-$sign-chains-position-top},        // #l-top-chains
      center $sign-chains-position-bottom,                            // #l-bottom-chains
      center center,                                                  // #l-sign-corners (diffuse and specular)
      center center,                                                  // #l-sign-horizontal-sides (diffuse and specular)
      center $sign-vertical-position-top,                             // #l-vertical-sides
      center center,                                                  // #l-sign-holes
      center center,                                                  // #l-horizontal-sides-depth
      center center,                                                  // #l-sign-corners-depth
      center center,                                                  // #l-bulletin-horizontal-sides-shadow
      center $sign-vertical-position-top,                             // #l-vertical-sides-shadow
      center center,                                                  // #l-sign-corners-shadow
      center center,                                                  // #l-sign-holes-shadow
      center center;                                                  // #l-sign-planks

    background-size: $sign-holes-width $sign-top-chains-height,       // #l-top-chains
      $sign-holes-width $sign-top-chains-height,                      // #l-bottom-chains
      100% $sign-board-height,                                        // #l-sign-corners (diffuse and specular)
      $sign-horizontal-width $sign-board-height,                      // #l-sign-horizontal-sides (diffuse and specular)
      100% $sign-vertical-height,                                     // #l-vertical-sides
      $sign-holes-width $sign-board-height,                           // #l-sign-holes
      $sign-horizontal-width $sign-board-height,                      // #l-horizontal-sides-depth
      100% $sign-board-height,                                        // #l-sign-corners-depth
      $sign-horizontal-width $sign-board-height,                      // #l-bulletin-horizontal-sides-shadow
      100% $sign-vertical-height,                                     // #l-vertical-sides-shadow
      100% $sign-board-height,                                        // #l-sign-corners-shadow
      $sign-holes-shadow-width $sign-board-height,                    // #l-sign-holes-shadow
      100% $sign-board-height;                                        // #l-sign-planks
```

Once we have our **block** and its two **modifiers** defined, it's time to create one instance for each board type in our HTML and then style them in CSS.

```html
<!DOCTYPE html>
<html lang="en">
<head>

  <!--  Meta  -->
  <meta charset="UTF-8" />
  <title>Medieval Board</title>

  <!--  Styles  -->
  <link rel="stylesheet" href="styles/index.processed.css">
</head>
<body>

  <div class='board board--type-bulletin bulletin'></div>
  <div class='board board--type-sign sign'></div>

</body>
</html>
```

As you can see, we created one `div` for each type of board and then added three classes to them:

`board` is the **block** entity while `board--type-bulletin` and `board--type-sign` are its **modifiers**. The last two classes, `bulletin` and `sign` are just and we instances of that **block** which we apply more specific properties to such as `heigh` and `width`. We will declare their rules in the `_examples` partial.

```scss
// EXAMPLES

.bulletin {
  height: $bulletin-board-height;
  left: 35%;
  margin-left: -30%;
  min-height: 30%;
  min-width: 40%;
  width: 60%;
}

.sign {
  height: $sign-board-height;
  right: 35%;
  margin-right: -30%;
  min-height: 30%;
  min-width: 15%;
  width: 25%;
}
```  

All there is left to do is creating a `div` for each of the board types in HTML.

```html
<!DOCTYPE html>
<html lang="en">
<head>

  <!--  Meta  -->
  <meta charset="UTF-8" />
  <title>Medieval Board</title>

  <!--  Styles  -->
  <link rel="stylesheet" href="styles/index.processed.css">
</head>
<body>

  <div class='board board--type-bulletin bulletin'></div>
  <div class='board board--type-sign sign'></div>

</body>
</html>
```

You should be getting something similar to this:

<p data-height="265" data-theme-id="0" data-slug-hash="EXjqRv" data-default-tab="result" data-user="andresangelini" data-pen-title="Medieval Board" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/EXjqRv/">Medieval Board</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

Have a look at the project's code [here][Project].

Wow! Isn't beautiful? It's truly amazing how far we have come since we were dreaming about making a responsive medieaval board with SVG stacks. I think we should take a well deserved rest becuase we still have some way to go. While we have achieved quite a lot and our medieval board is actually responsive, we still have to make sure it works properly across browsers and apply the necesarry measures when it doesn't.


[BEM]: http://getbem.com/
[Sass Ampersand]: https://css-tricks.com/the-sass-ampersand/
[Project]: https://codepen.io/andresangelini/project/editor/Aarxxz
