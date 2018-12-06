This post is just one part of a larger story about an alchemist's personal quest for **Making a Responsive Medieval Board With SVG Stacks**. You may read the chapters in any order you want but I would otherwise suggest you to do it in the proper order to get the full context of the project.

## Chapters

1. [Making a Responsive Medieval Board With SVG Stacks][ch-1]
2. [Making an SVG Stretch][ch-2]
3. [Inverting SVG Elements][ch-3]
4. [Rotating SVG Elements][ch-4]
5. [Clipping SVG Elements][ch-5]
6. [Improving Organization With Defs][ch-6]
7. [Namespacing][ch-7]
8. [Styling an SVG][ch-8]
9. [Using SVG patterns][ch-9]
10. [Clipping SVG Elements With Complex Shapes][ch-10]
11. [Using an SVG stack With Fragment Identifiers][ch-11]
12. [Getting Sassy With The Board][ch-12]
13. [When Sass Met BEM][ch-13]
14. [Browser Support For SVG Stacks][ch-14]
15. [Sass Mixins For SVG Stacks][ch-15]
16. [Combining Modernizr Detects][ch-16]
17. [Tiling With Sass][ch-17]
18. [Error Handling in Sass][ch-18]

## To recap

In the [last post][ch-14] we created a new CSS rule for those scenarios where SVGs don't scale properly when used as `background-image` and the only browser affected by this particular issue was Internet Explorer up to version 10. Sadly, Modernizr doesn't have any module for detecting support for SVG as `background-image`, though. However, by carefully checking out [Can I Use] we discovered that Internet Explorer 10 and older do not support CSS [pointer-events] either, which does have a module, so we used that instead.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {

    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {

    }
  }
}
```

## A workaround for missing Modernizr modules

The root of the problem seems to be that those versions of IE don't like using SVG in multiple `background-image`s. A way around that, then, is using some JavaScript magic to split the board into many `div`s, each with a single `background-image` pointing to its respective SVG fragment instead.

Let's start off by creating a simple JavaScript conditional to make the change if CSS [pointer-events] are supported. Will do this in a new file called `index.js` living inside the `scripts` we created earlier right alongside `modernizr.js`.

```javascript
if (Modernizr.csspointerevents) {
  // supported
  appendElementsToBoard('div');
} else {
  // not-supported

}

function appendElementsToBoard(elem) {
  // IE 8 raises an error with document.getElementsByClassName().
  try {
    // Insert multiple elements in every board div.
    for (var i = 0; i < document.getElementsByClassName('board').length; i++) {
      for (var k = 1; k < 14; k++) {
       document.getElementsByClassName('board')[i].appendChild(document.createElement(elem));   
      }
    }
  } catch(err) {
    // do nothing.
  }
}
```

Instead of doing all that stuff right inside the conditional, it's wise to turn it into a function, which in this case I called `appendElementsToBoard`. The function in itslelf isn't anything special and should be quite self-explanatory. The only worth thing mentioning, though, is than since the `div`s will need to be absolutely positioned to have their own sizes and positions, we will have to insert them inside the original `div` instead of replacing it.

Now let's go back to the `_board.scss` partial and make sure they all have their **position** as **absolute**.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }
    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }
    }
  }
}
```

Again, the `@at-root` Sass directive let us specify that we want all child `div`s of `.board--type-bulletin` and `.board--type-sign` to have `position: absolute` only when `.no-csspointerevents` is added to the `html` element by writing this rule inside the ones for `.board--type-bulletin` and `.board--type-sign`.

If at this point we were to write our rules in plain CSS we would soon find ourselver doing some ardous and repetetive task. Let's try with the `planks` first to see how this would be like:

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }

      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(1) {
        background-image: url("#{$path-to-board}#l-bulletin-planks");
        bottom: 0;
        height: $bulletin-board-height;
        width: 100%;
      }
    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }

      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(1) {
        background-image: url("#{$path-to-board}#l-sign-planks");
        bottom: $sign-board-position-top;
        height: $sign-board-height;
        width: 100%;
      }
    }
  }
}
```

## Sass Mixins

Ugh! What a mess that is. I hope you can see how much repetition we can avoid here. Both `.board-type-bulletin` and `.board--type-sign` have declarations for the same properties but with different values except the `with: 100%`, so let's throw some Sass [mixin] magic to the mix to fix this issue without repeating ourselves.

```scss
@mixin planks($path, $board-height, $board-position-top) {
  background-image: url($path);
  bottom: $board-position-top;
  height: $board-height;
  width: 100%;
}
```
**Mixins** are basically **funcions** that can take **arguments** but can only return **declarations**. In this case, we declare a mixin by using the `@mixin` directive, name it as `planks` and stablish three parameter for it between parenthesis:

- `$path`: the URL path of the SVG fragment to pass to the `background-image`.
- `board-height`: the `height` of the board.
- `board-position-top`: the `top` position of the board.

By using our newly created **mixin** our previous rules becomes much more readable by turning four lines of code into just one.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }

      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(1) {@include planks("#{$path-to-board}#l-bulletin-planks", $bulletin-board-height, 0);}
    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div:nth-child(1) {@include planks("#{$path-to-board}#l-sign-planks", $sign-board-height, $sign-board-position-top);}
    }
  }
}
```

The same can be done for each of the `div`s by creating more **mixins** in the `_mixins.scss` partial.

```scss
// A mixin for centering absolute positioned elements.
@mixin center-horizontally($width) {
  margin-left: calc(#{$width} / (-2));
  margin-left: calc(#{strip-calc($width)} / (-2));    // For browsers that don't support nested calc() functions.
  left: 50%;
  width: #{$width};
}

@mixin chains($path, $holes-width, $chains-height, $chains-position-top) {
  @include center-horizontally($holes-width);
  background-image: url($path);
  height: $chains-height;
  top: #{-$chains-position-top};
}

@mixin plaque($path) {
  @include center-horizontally($bulletin-plaque-width);
  background-image: url($path);
  bottom: 0;
  height: $bulletin-board-height;
}

@mixin corners($path, $board-height, $board-position-top) {
  background-image: url($path);
  bottom: $board-position-top;
  height: $board-height;
  width: 100%;
}

@mixin horizontal-sides($path, $horizontal-width, $board-height, $board-position-top) {
  @include center-horizontally($horizontal-width);
  background-image: url($path);
  bottom: $board-position-top;
  height: $board-height;
}

@mixin vertical-sides($path, $vertical-height, $board-height, $side-position-left, $pairs-of-chains) {
  background-image: url($path);
  height: $vertical-height;
  top: calc((100% - #{$board-height}) / #{$pairs-of-chains} + #{strip-calc(#{$side-position-left})});
  width: 100%;
}

@mixin holes($path, $holes-width, $board-height, $board-position-top) {
  @include center-horizontally($holes-width);
  background-image: url($path);
  bottom: $board-position-top;
  height: $board-height;
}

@mixin planks($path, $board-height, $board-position-top) {
  background-image: url($path);
  bottom: $board-position-top;
  height: $board-height;
  width: 100%;
}
```

**Mixins** can be **nested** inside other **mixins** as you can see above, which comes as a really nice feature to create more generic **mixins** such as our `center-horizontally` one. Will be using this one quite often each time we want to center an absolutely positioned element.

Thanks to those **mixins** we can dedicate only one line for each `div`.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      & div {
        position: absolute;
      }

      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(13) {@include plaque("#{$path-to-board}#l-plaque");}
      & div:nth-child(12) {@include chains("#{$path-to-board}#l-top-chains", $bulletin-holes-width, $bulletin-chains-height, $bulletin-chains-position-top);}
      & div:nth-child(11) {@include corners("#{$path-to-board}#l-bulletin-corners", $bulletin-board-height, 0);}
      & div:nth-child(10) {@include horizontal-sides("#{$path-to-board}#l-bulletin-horizontal-sides", $bulletin-horizontal-width, $bulletin-board-height, 0);}
      & div:nth-child(9) {@include vertical-sides("#{$path-to-board}#l-vertical-sides", $bulletin-vertical-height, $bulletin-board-height, $bulletin-side-position-left, 1);}
      & div:nth-child(8) {@include holes("#{$path-to-board}#l-bulletin-holes", $bulletin-holes-width, $bulletin-board-height, 0);}
      & div:nth-child(7) {@include horizontal-sides("#{$path-to-board}#l-horizontal-sides-depth", $bulletin-horizontal-width, $bulletin-board-height, 0);}
      & div:nth-child(6) {@include corners("#{$path-to-board}#l-bulletin-corners-depth", $bulletin-board-height, 0);}
      & div:nth-child(5) {@include horizontal-sides("#{$path-to-board}#l-horizontal-sides-shadow", $bulletin-horizontal-width, $bulletin-board-height, 0);}
      & div:nth-child(4) {@include vertical-sides("#{$path-to-board}#l-vertical-sides-shadow", $bulletin-vertical-height, $bulletin-board-height, $bulletin-side-position-left, 1);}
      & div:nth-child(3) {@include corners("#{$path-to-board}#l-bulletin-corners-shadow", $bulletin-board-height, 0);}
      & div:nth-child(2) {@include holes("#{$path-to-board}#l-bulletin-holes-shadow", $bulletin-holes-shadow-width, $bulletin-board-height, 0);}
      & div:nth-child(1) {@include planks("#{$path-to-board}#l-bulletin-planks", $bulletin-board-height, 0);}
    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {
      /* Instead of using a single div with multiple background images, use multiple divs each with their own background image pointing to its respective layer in the SVG stack. */
      & div:nth-child(13) {@include chains("#{$path-to-board}#l-top-chains", $sign-holes-width, #{calc-chains-height($sign-board-height, $sign-chains-position-top, 2)}, $sign-chains-position-top);}
      & div:nth-child(12) {
        @include chains("#{$path-to-board}#l-bottom-chains", $sign-holes-width, #{calc-chains-height($sign-board-height, $sign-chains-position-top, 2)}, $sign-chains-position-top);
        top: $sign-bottom-chains-div-position-top; // Overwrite function's heihgt.
      }
      & div:nth-child(11) {@include corners("#{$path-to-board}#l-sign-corners", $sign-board-height, $sign-board-position-top);}
      & div:nth-child(10) {@include horizontal-sides("#{$path-to-board}#l-sign-horizontal-sides", $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
      & div:nth-child(9) {@include vertical-sides("#{$path-to-board}#l-vertical-sides", $sign-vertical-height, $sign-board-height, $sign-side-position-left, 2);}
      & div:nth-child(8) {@include holes("#{$path-to-board}#l-sign-holes", $sign-holes-width, $sign-board-height, $sign-board-position-top);}
      & div:nth-child(7) {@include horizontal-sides("#{$path-to-board}#l-horizontal-sides-depth", $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
      & div:nth-child(6) {@include corners("#{$path-to-board}#l-sign-corners-depth", $sign-board-height, $sign-board-position-top);}
      & div:nth-child(5) {@include horizontal-sides("#{$path-to-board}#l-horizontal-sides-shadow", $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
      & div:nth-child(4) {@include vertical-sides("#{$path-to-board}#l-vertical-sides-shadow", $sign-vertical-height, $sign-board-height, $sign-side-position-left, 2);}
      & div:nth-child(3) {@include corners("#{$path-to-board}#l-sign-corners-shadow", $sign-board-height, $sign-board-position-top);}
      & div:nth-child(2) {@include holes("#{$path-to-board}#l-sign-holes-shadow", $sign-holes-width, $sign-board-height, $sign-board-position-top);}
      & div:nth-child(1) {@include planks("#{$path-to-board}#l-sign-planks", $sign-board-height, $sign-board-position-top);}
    }
  }
}
```

Done! Now the board should switch from using only one `div` with multiple `background-image`s to multiple `div`s with a single `background-image` pointing to a specefic SVG fragment whenever SVGs used in `multiple-image`s are not supported by the browser.

Let's take a breather and leave solving the rest of the issues for the [next post][ch-16] where will introduce a more advanced technique for feature detecttion.



[ch-1]: ../Making-a-responsive-medieval-board-with-SVG-stacks
[ch-2]: ../Making-an-SVG-stretch
[ch-3]: ../Inverting-SVG-elements
[ch-4]: ../Rotating-SVG-elements
[ch-5]: ../Clipping-SVG-elements
[ch-6]: ../Improving-organization-with-defs
[ch-7]: ../Namespacing
[ch-8]: ../Styling-an-SVG
[ch-9]: ../Using-SVG-patterns
[ch-10]: ../Clipping-SVG-elements-with-complex-shapes
[ch-11]: ../Using-an-SVG-stack-with-fragment-identifiers
[ch-12]: ../Getting-Sassy-with-the-board
[ch-13]: ../When-Sass-met-BEM
[ch-14]: ../Browser-support-for-SVG-stacks
[ch-15]: ../Sass-mixins-for-SVG-stacks
[ch-16]: ../Combining-modernizr-detects
[ch-17]: ../Tiling-with-Sass
[ch-18]: ../Error-handling-in-Sass
[Can I Use]: https://caniuse.com/
[pointer-events]: https://caniuse.com/#feat=pointer-events
[mixin]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#defining_a_mixin
