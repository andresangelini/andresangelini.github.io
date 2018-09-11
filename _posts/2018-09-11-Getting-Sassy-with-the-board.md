### Assembling the board from an SVG with Sass

Much like when we found ourselves with a really messy SVG structure and decided to organize it using `<defs>`, namespacing and some styling, we find ourselves once again in a little bit of a problem. As we have stablished in our design, we have a **bulletin board** and **sign-board**, which means there will be a myriad of variables, modifications, and edge cases that our CSS will have to handle. If your aren't careful, we can end up with a truly mantainance nightmare.

This makes the perfect scenario for introducing [Sass] into our project. This tool is a CSS preprocessor which helps us write better style sheets by letting us use features like variables and functions that don't exits in vanilla CSS. One of those features is the hability to modularize our CSS into so called "partials" to make things easier to mantain. "Partials" are files where you can put different snippets of code which Sass will process and output into a single CSS file.

After going through [Sass Basics] and [installing] it, we create the following files in a `styles` folder.

```
/styles
  _boards.scss
  _examples.scss
  _functions.scss
  _mixins.scss
  _variables.scss
  _index.scss
```

And then we set the `@imports` in the `_index.scss` partial.

```scss
// IMPORTS

@import 'functions';
@import 'variables';
@import 'mixins';
@import 'boards';
@import 'examples';
```

[Sass] will add (or overwrite) the processed CSS output as a file called `index.processed.scss` in the same `styles` folder when you use the `sass --watch` command (or any other name you like if you build it manually with `sass index.scss [somename].css`.

#### Sass variables

Before we even begin writing any CSS, we should store values like dimensions, colors, and URLs in their own variables. Since we have just created the SVG file, why don't we start with that?

Let's open the `viariables.scss` and write:

```scss
// URL paths.
$path-to-board: "https://cdn.rawgit.com/andresangelini/d57f93b605eec432fdea98b969aaac72/raw/f0ee9369b781dcf01eefc37575a99d8e5dbe7b35/board.svg";
```

By the way, if you are wondering how to serve and SVG like that, just check out [this excellent article] at SitePoint.

As for the colors, we will only need to store one since all the others are already defined in the SVG itself. The reason for this will be explained later on. Anyways, even if it's only one, it's still a good practice to name colors with a **descriptive name** as well as wtih a **functional name**. The first one can be anything you like. You can be creative or, if you are like me and have difficulty coming up with names, go [here][name that color] and have some fun!

```scss
  // Base colors.
  $color-cape-palliser                  : #a2703f;

  // Elements colors.
  $color-planks-diffuse                 : $color-cape-palliser;
```

You can see here that I first named the color in a **descriptive** way as `cape-palliser`. Then I stored this new variables as the value of a second one named in a **functional** way as `planks-diffuse` since that's where is going to be used. Both variables were appended with `color` for an easier read.

Next are all the variables related to the elements dimensions and positions. Remember that we have two types of board (**bulletin** and **sign**) which share some variables in common but also have many that are unique to each. We will start off by declaring those that can be modified by the user first and then those which should not be modified but are shared in common between the two types of board.

```scss
// Change these values to your liking. The length of the chains will adjust automatically.
$bulletin-board-height                : 95%; // 95%
$bulletin-plaque-width                : 35%;
$sign-board-height                    : 90%;

// These values are common to both types of boards. DO NOT CHANGE unless you modified the SVG graphics.
$board-depth                          : 6.5px;
$shadow-width                         : 2.16px;
$side-overlap                         : 9.4px;
$side-curve-left                      : 5px;
$side-curve-right                     : calc(100% - #{$side-curve-left});
$grooves-height                       : 527px;
$grooves-width                        : 568px;
$shade-height                         : 308px;
$chain-link-height                    : 45.94px;
```

If you look at `$side-curve-right` you can see we can make calculation with variables just like with any other programming language. However, we did something else in order for it to work. The **hash** symbol and the **curly brackets** wrapping the variable is called [interpolation] and it bassically let us make sure that a variable will be treated as plain CSS instead of a string. In other words:

```scss
$number: 20px;

div {
  width: calc(100% - $number);  
}
```

Will output:

```scss
div {
  width: calc(100% - "20px");
}
```

While:

```scss
$number: 20px;

div {
  width: calc(100% - #{$number});  
}
```

Will correctly result in:

```scss
div {
  width: calc(100% - 20px);
}
```

With this new knowledge about what [interpolation] is and how to use it, we can now move on to those variables that shouldn't be modified by the user and are unique to each type of board.

```scss
// Measurements for 'bulletin' type board. DO NOT CHANGE unless you modified the SVG graphics.
$bulletin-corner-width                : 56px;
$bulletin-chains-position-top         : 22px;

// Measurements for 'sign' type board. DO NOT CHANGE unless you modified the SVG graphics.
$sign-corner-width                    : 29.78px;
$sign-vertical-position-top           : calc(50% - 3px); // Remember: background position in perecentage works differently.
$sign-chains-position-top             : 22px;
$sign-chains-position-bottom          : calc(100% + #{$sign-chains-position-top});
$sign-bottom-chains-position-top      : calc(#{$sign-chains-position-top} / 2);
```

#### Sass functions

The values so far are either static or their calculations are pretty unique (meaning, we won't be reusing them). However, the ones we are going to set up next will share some kind of calculation in which a Sass function would come very in handy. Let's take the `x` position of the horizontal sides of the board's frame as an example.

```scss
$bulletin-side-position-left: calc(#{$bulletin-corner-width} - #{$side-overlap});
```

Where `$corner-width` is the `width` of the **corner** piece of the board and `$side-overlap` is the amount of **overlap** between the **corners** and the **horizontal sides of the frame**.

This is basically the same formula for the **sign board** except that the **width** of its **corners** is different.

```scss
$sign-side-position-left    : calc(#{$sign-corner-width} - #{$side-overlap});
```

So instead of repeating ourselves, let's create the following function in the `_functions.scss` partial.

```scss
@function calc-side-position-left($corner-width) {
  @return calc(#{$corner-width} - #{$side-overlap});
}
```
Note that I prepended `calc` to the name function to denote that this functions `return`s a CSS `calc()` function. Also note that CSS variables need to be [interpolated][interpolation] by placing them inside curly brackets after a hash as in `#{$var}` as we've learned earlier. Now we can create those same variables using this function instead.

```scss
$bulletin-side-position-left: calc-side-position-left($bulletin-corner-width);
$sign-side-position-left    : calc-side-position-left($sign-corner-width);
```

Rinse and repeat for the next funcion:

```scss
@function calc-side-position-bottom($corner-width) {
  @return calc((100% - (#{$corner-width} - #{$side-overlap})) - #{$board-depth});
}
```

Let's now create a function for calculating the width of the frame horizontal sides.

```scss
@function calc-horizontal-width($corner-width) {
  @return calc(100% - #{calc-side-position-left(#{$corner-width})} * 2);
}
```

However, if we try to use it like this, it's just not going to work because **you can't nest interpolations**. When you use functions inside a `calc()` function, **their arguments should not be interpolated**. So, the right way to write that function is:

```scss
@function calc-horizontal-width($corner-width) {
  @return calc(100% - #{calc-side-position-left($corner-width)} * 2);
}
```

But there is yet another problem with this if you care about backward compatibility with older browsers and that is that some of them do not support nested `calc()`s. It would be a good idea then to create a simple function that strips the `calc` leaving just the argument with parethesis included **only if** said argument is actually a `calc()`. Since **functions should do only one thing and do it well**, we will turn this condition into a function in its own which will run this check and return a boolean by comparing the four first characters of the stringified argument to the string `"calc"`.

```scss
// Checks if a value is actually a calc() function.
@function is-calc($value) {
  @return str_slice("#{$value}", 0, 4) == "calc";
}
```

With that done, we can now create the function that will strip de `calc` from the name of the `calc()` function by first running the check we have just created. If it is, it will do its job by turning the argument into a `string`, removing the `"calc"` with Sass built-in `str_slice()` function and returning the unquoted result with `unquote()`, which also comes with Sass. If on the other hand, the argument is not `calc()`, the function will simple return its argument as it is.

```scss
// Strips the "calc" part from a calc() function or returns the argument if it's
// present.
@function strip-calc($calc) {
  @if (is-calc($calc)) {
    @return unquote(str_slice("#{$calc}", 5, -1));
  } @else {
    @return $calc;
  }
}
```

With these two new tools in our toolbox we can go ahead and create all the functions needed for the rest of the variables. The actual formulas should be pretty self explanatory so we won't go deeper into those.

```scss
@function calc-horizontal-width($corner-width) {
  @return calc(100% - #{strip-calc(calc-side-position-left($corner-width))} * 2);
}
@function calc-board-position-top($board-height, $pairs-of-chains) {
  @return calc((100% - #{$board-height}) / #{$pairs-of-chains});
}
@function calc-vertical-height($corner-width, $board-height) {
  @return calc(#{$board-height} - #{strip-calc(calc-side-position-left($corner-width))} * 2 - #{$board-depth});
}
@function calc-vertical-position-top($corner-width) {
  @return calc(100% - #{strip-calc(calc-side-position-left($corner-width))} - #{$board-depth});
}
@function calc-chains-height($board-height, $chains-position-top, $pairs-of-chains) {
  @return calc(#{strip-calc(calc-board-position-top($board-height, $pairs-of-chains))} + #{$chains-position-top} * 2);
}
@function calc-holes-width($corner-width) {
  @return calc(#{strip-calc(calc-horizontal-width($corner-width))} * 0.8);
}
@function calc-holes-shadow-width($corner-width) {
  @return calc(#{strip-calc(calc-holes-width($corner-width))} + #{$shadow-width} * 2);
}
@function calc-holes-position-left($corner-width) {
  @return calc((100% - (#{strip-calc(calc-horizontal-width($corner-width))} * 0.8)) / 2);
}
@function calc-holes-position-right($corner-width) {
  @return calc(100% - ((100% - (#{strip-calc(calc-horizontal-width($corner-width))} * 0.8)) / 2));
}
@function calc-corner-left($corner-width) {
  @return calc(#{strip-calc(calc-side-position-left($corner-width))} - 5px);
}
@function calc-corner-right($corner-width) {
  @return calc(#{strip-calc(calc-side-position-right($corner-width))} + 5px);
}
```

All we have to do now is to invoke our functions with the right arguments to declare the rest of the variables in the `_variables.scss` partial grouping them by board type.

```scss
// Measurements for 'bulletin' type board. DO NOT CHANGE unless you modified the SVG graphics.
$bulletin-corner-width                : 56px;
$bulletin-side-position-left          : calc-side-position-left($bulletin-corner-width);
$bulletin-side-position-right         : calc-side-position-right($bulletin-corner-width);
$bulletin-side-position-bottom        : calc-side-position-bottom($bulletin-corner-width);
$bulletin-horizontal-width            : calc-horizontal-width($bulletin-corner-width);
$bulletin-vertical-height             : calc-vertical-height($bulletin-corner-width, $bulletin-board-height);
$bulletin-vertical-position-top       : calc-vertical-position-top($bulletin-corner-width);
$bulletin-chains-position-top         : 22px;
$bulletin-chains-height               : calc-chains-height($bulletin-board-height, $bulletin-chains-position-top, 1);
$bulletin-holes-width                 : calc-holes-width($bulletin-corner-width);
$bulletin-holes-shadow-width          : calc-holes-shadow-width($bulletin-corner-width);

// Measurements for 'sign' type board. DO NOT CHANGE unless you modified the SVG graphics.
$sign-corner-width                    : 29.78px;
$sign-side-position-left              : calc-side-position-left($sign-corner-width);
$sign-side-position-right             : calc-side-position-right($sign-corner-width);
$sign-side-position-bottom            : calc-side-position-bottom($sign-corner-width);
$sign-horizontal-width                : calc-horizontal-width($sign-corner-width);
$sign-board-position-top              : calc-board-position-top($sign-board-height, 2);
$sign-vertical-height                 : calc-vertical-height($sign-corner-width, $sign-board-height);
$sign-vertical-position-top           : calc(50% - 3px); // Remember: background position in perecentage works differently.
$sign-chains-position-top             : 22px;
$sign-chains-position-bottom          : calc(100% + #{$sign-chains-position-top});
$sign-top-chains-height               : calc-chains-height($sign-board-height, $sign-chains-position-top, 2);
$sign-bottom-chains-position-top      : calc(#{$sign-chains-position-top} / 2);
$sign-bottom-chains-div-position-top  : calc(100% + 22px - #{strip-calc($sign-top-chains-height)});
$sign-bottom-chains-height            : calc-chains-height($sign-board-height, #{strip-calc($sign-bottom-chains-position-top)}, 2);
$sign-holes-width                     : calc-holes-width($sign-corner-width);
$sign-holes-position-left             : calc-holes-position-left($sign-corner-width);
$sign-holes-position-right            : calc-holes-position-right($sign-corner-width);
$sign-holes-shadow-width              : calc-holes-shadow-width($sign-corner-width);
```

Again, these variables should not change unless something in the SVG is modified. But if we do find the need to do any modification to our graphic, we will only have to change the functions arguments insntead of having to rewrite each calculation manually.

This should be the perfect moment to take a well deserved rest and let all we have seen and done so far sink in. In the next stage, we will finally jump into making the board itself.

[Sass]: https://sass-lang.com/
[Sass Basics]: https://sass-lang.com/guide
[installing]: http://Sass-lang.com/install
[this excellent article]: https://www.sitepoint.com/why-hosting-your-svgs-is-hard-and-how-to-beat-it/
[name that color]: http://chir.ag/projects/name-that-color/
[interpolation]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#interpolation_
