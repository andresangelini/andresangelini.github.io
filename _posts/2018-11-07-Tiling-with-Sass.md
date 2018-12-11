This post is just one part of a larger story about an alchemist's personal quest for **Making a Responsive Medieval Board With SVG Stacks**. You may read the chapters in any order you want but I would strongly suggest following the proper order to get the full context of the project.

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

## An otherwise mess of declarations

As briefly mentioned at the end of the [last post][ch-16], it would be an unimaginably tedious task, to say the least, if you had to remake an SVG pattern using multilpe `background-image`s in vanilla CSS. Just to give you an idea of how incredibly messy this would be, have a look at the line `303` of [`index.processed.css`] where the `background-image`s for the `planks` are set. If you are thinking a **mixin** could solve the problem, think again. **Mixins** can only return **declarations** and we don't want to create multiple declarations for `background-image` because we would only be overriding the previous one instead of setting multiple ones. To understand better what we need to do, let's recap how each SVG pattern is made.

The planks have actually three separate layers: a plain **background color**, the **grooves** of the wood and their **shades**. Each of them behave differently but all of them must be **clipped** so that they don't overflow through the wooden frame. Basically, as the board expands and contracts, each layer should:

- **background color**: fill all available space.
- **grooves**: repeat both horizontally and vertically.
- **shades**: have a simetric copy on each side of the board, both repeating horizontally and vertically.

Let's start with what seems easier; the **background color**.

## Clipping in CSS

Safari 11 might not support SVG patterns or `background-repeat` with alpha but it does have partial support for CSS `clip-path` if you use it along with CSS [shapes]. These last ones are a relatively new feature that allow us to draw simple figures in CSS. You have your basic shapes such as `line()`, `rectangle()` and `circle()`. To create a figure using both you only need to set a `clip-path` **property** with any of these shapes as a **value** to a `div`. For example:

```css
clip-path: circle(50px);
```

What we are actually doing, in fact, is creating a regular squared `div` and clipping it with the shape we want it to have, as shown below.

<p data-height="265" data-theme-id="0" data-slug-hash="PypmBO" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Simple CSS shape" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/PypmBO/">Simple CSS shape</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

But you can also build more complex figures using `polygon()`, which is what we are going to use to reproduce our clipped SVG pattern.

<p data-height="265" data-theme-id="0" data-slug-hash="vpYbPP" data-default-tab="result" data-user="andresangelini" data-pen-title="Clipping SVG elements with complex shapes" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/vpYbPP/">Clipping SVG elements with complex shapes</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

To draw a shape using `polygon()`, we once again set it as the **value** of the `clip-path` **property** with the `x y` coordinates of each point as its arguments starting from the **top left corner** and going **clockwise**.

```css
cip-path: polygon(x-1 y-1, x-2 y-2, x-3 y-3, ...);
```
<p data-height="265" data-theme-id="dark" data-slug-hash="JmWJKE" data-default-tab="result" data-user="andresangelini" data-pen-title="CSS polygon" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/JmWJKE/">CSS polygon</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

All the usual units are allowed which is exactly what we need in order for this `clip-path` to be flexible. However, the boards differ not only in the size of their corners but the sign type also has holes in the bottom, so it would be wise to write a function that returns a `polygon()` depending on which board it is. To do this, we will need to use another great feature of Sass; the built-in [`if()`][if] function (not to be confused with the [`@if`][if] directive).

It basically works like this:

```
if(true, a, b) returns a
if(false, a, b) returns b
```

Which in our case would be:

```scss
if($corner-width == $sign-corner-width, $bottom-holes, null)
```

Where `$bottom-holes` represents the coordinates of the points drawing the section of the clip where the holes should be.

```scss
$bottom-holes: calc-holes-position-right($corner-width) $side-curve-right + "," +                        /* B holes BR */
               calc-holes-position-right($corner-width) calc-side-position-bottom($corner-width) + "," + /* B holes TR */
               calc-holes-position-left($corner-width) calc-side-position-bottom($corner-width) + "," +  /* B holes TL */
               calc-holes-position-left($corner-width) $side-curve-right + ",";
```

The result is a function that does everything for us by simply changing its argument to the correspondant `$corner-width`.

```scss
@function planks-clip($corner-width) {
  // Sign board has bottom holes while bulletin board hasn't.
  $bottom-holes: calc-holes-position-right($corner-width) $side-curve-right + "," +                        /* B holes BR */
                 calc-holes-position-right($corner-width) calc-side-position-bottom($corner-width) + "," + /* B holes TR */
                 calc-holes-position-left($corner-width) calc-side-position-bottom($corner-width) + "," +  /* B holes TL */
                 calc-holes-position-left($corner-width) $side-curve-right + ",";                          /* B holes BL */

  @return polygon(0 calc-side-position-right($corner-width),                                               /* BL corner 3 */
                  $side-curve-left 50%,                                                                    /* L side center */
                  0 $corner-width,                                                                         /* TL corner 1 */
                  calc-corner-left($corner-width) calc-corner-left($corner-width),                         /* TL corner 2 */
                  calc-side-position-left($corner-width) 0,                                                /* TL corner 3 */
                  calc-holes-position-left($corner-width) $side-curve-left,                                /* Top holes TL */
                  calc-holes-position-left($corner-width) calc-corner-left($corner-width),                 /* Top holes BL */
                  calc-holes-position-right($corner-width) calc-corner-left($corner-width),                /* Top holes BR */
                  calc-holes-position-right($corner-width) $side-curve-left,                               /* Top holes TR */
                  calc-side-position-right($corner-width) 0,                                               /* TR corner 1 */
                  calc-corner-right($corner-width) calc-corner-left($corner-width),                        /* TR corner 2 */
                  100% calc-side-position-left($corner-width),                                             /* TR corner 3 */
                  $side-curve-right 50%,                                                                   /* R side center */
                  100% calc-side-position-right($corner-width),                                            /* BR orner 1 */
                  calc-corner-right($corner-width) calc-corner-right($corner-width),                       /* BR corner 2 */
                  calc-side-position-right($corner-width) $side-curve-right,                               /* BR corner 3 */
                  if($corner-width == $sign-corner-width, $bottom-holes, null)                             /* B holes BL */
                  calc-side-position-left($corner-width) $side-curve-right,                                /* BL corner 1 */
                  calc-corner-left($corner-width) calc-corner-right($corner-width)                         /* BL corner 2 */
                  );
}
```

Check out the Pen bellow and play around with `$corner-width` argument in the `planks-clip()` function of each board type's `clip-path` property. Try replacing `$bulleting-corner-width` with `$sign-corner-width` in for the **bulletin** type and vice versa for the **sign** type.

<p data-height="315" data-theme-id="dark" data-slug-hash="NOjXbW" data-default-tab="css,result" data-user="andresangelini" data-pen-title="CSS clip-path with polygon()" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/NOjXbW/">CSS clip-path with polygon()</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## Making patterns in plain CSS

A very basic way to simulate the grooves pattern of the planks in vanilla CSS, for example, would be to do something along these lines:

```css
& div:nth-child(1) {
  background-image: url("grooves.svg"),
                    url("grooves.svg"),
                    url("grooves.svg"),
                    url("grooves.svg"),
                    url("grooves.svg"),
                    url("grooves.svg");
  background-position: 0,
                       568px 0,
                       1136px 0,
                       0 568px,
                       568px 568px,
                       1136px 568px;
  background-size: 100%;
}
```

<p data-height="265" data-theme-id="dark" data-slug-hash="LgjQKo" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with plain CSS" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/LgjQKo/">Tiled background with plain CSS</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

The intention in this example is to create a tile pattern out of the grooves so that there are a total of 6 tiles next to each other in both `x` and `y` axis. But of course, we already know this could be a lot better by doing it the Sass way.

## Multiple `background-image`s with Sass

First off, let's make a simple function for creating multiple `background-image`s out from a single file. It should have two parameters; the `$path` URL to the image and the `$amount` of copies we want. A [`@for`][for] loop will do the trick here.

```scss
// Returns multiple backgroun images.
@function tiling-images($path, $amount) {
  $result: url($path);

  @for $i from 1 to $amount {
    $result: $result, url($path);
  }

  @return $result;
}
```

<p data-height="265" data-theme-id="dark" data-slug-hash="wYYJpQ" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Multiple backgrounds with Sass" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/wYYJpQ/">Multiple backgrounds with Sass</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

It's important to remember that, contrary to what one would normally expect, the first `background-image` is the one closest to the user and the ones than come after it are placed behind.

## Multiple `background-position`s with Sass

The `background-position` property let us move a `background-image` in the `x` and `y` directions. In our earlier example we set the three first images next to each other along the `x` axis and the three last ones in a second row below.

<p data-height="265" data-theme-id="dark" data-slug-hash="LgjQKo" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with plain CSS" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/LgjQKo/">Tiled background with plain CSS</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

In other words, the positions of the images can be described by the general ecuation of a straight line `$tile-dx * $i + $line-x` for `x` axis and `$tile-dy * $i + $line-y` for `y`, where:

- `$tile-dx` and `$tile-dy`are the horizontal and vertical distances between each `background-image`.
- `$i` is the `@for` loop **iterator** used to increase the distance of the **tiles** from the container's top left corner.
- `$line-x` and `$line-y` are the coordinates of the **line** of `background-image`'s top left corner.

With this, we will create a new function called `tiling-line-positions` which will loop `$tiles` number of times to concatenate each new `$result` to the previous one as `$i` increases and returns multiple `background-postion`'s in a **line** pattern.

```scss
@function tiling-line-positions($line-x: 0px, $line-y: 0px, $tile-dx: 0px, $tile-dy: 0px, $tiles: 1) {
  $result: null;

  @for $i from 0 to $tiles {
    $result: $result, calc((#{$tile-dx} * #{$i}) + #{$line-x}) +
                      " " +
                      calc((#{$tile-dy} * #{$i}) + #{$line-y});
  }

  @return $result;
}
```

Now we must use the same logic to obtain a **tiled** background in **two** directions by creating multiple **lines** of **tiles**. This time, the ecuations change to `$line-dx * $i + $tiling-x` and `$line-dy * $i + $tiling-y` for each `$line-x` and `$line-y` where:

- `$line-dx` and `$line-dy` are the horizontal and vertical distances between each line pattern.
- `$i` is the `@for` loop **iterator** used to increase the distance of the **lines of tiles** from the container's top left corner.
- `$tiling-x` and `$tiling-y` are the coordinates of the tiling's top left corner.

The idea behind this is to create a **tiling** out of **lines** using the `tiling-line-positions()` function we just created and set the coordinates for each **line** as the **iteraror** increases so we don't have to write the same function over and over again to get more than one line of `background-position`:

```
background-positions: tiling-line-positions($line-x: 0px,
                                            $line-y: 0px,
                                            $tile-dx: 568px,
                                            $tile-dy: 0px,
                                            $tiles: 3),
                      tiling-line-positions($line-x: 0px,
                                            $line-y: 568px,
                                            $tile-dx: 568px,
                                            $tile-dy: 0px,
                                            $tiles: 3);
```

This positions 3 images in a row next to each other without any gap in between and another 3 right below them also in a line. However, our new function called `tiling-positions` will make writing this much easier. It will be an improvement over `background-positions-in-a-line` by using it as a base to work upon, which means there won't be any reason to use it outside `tiling-positions()` anymore. That's why will append an **underscore** to its name to let other developers know that it's **private** and should not be used anywhere else: `_tiling-line-positions()`. With that out of the way, we can move on to creating our new function.

```scss
@function tiling-positions($tiling-x: 0px,
                           $tiling-y: 0px,
                           $tile-dx: 0px,
                           $tile-dy: 0px,
                           $line-dx: 0px,
                           $line-dy: 0px,
                           $tiles-per-line: 1,
                           $lines: 1) {
  $result: null;

  @for $i from 0 to $lines {
    $result: $result, _tiling-line-positions(calc((#{$line-dx} * #{$i}) + #{$tiling-x}), calc((#{$line-dy} * #{$i}) + #{$tiling-y}), $tile-dx, $tile-dy, $tiles-per-image);
  }

  @return $result;
}
```

And we can set the `background-positions` for the `grooves` like this:

```
background-position: tiling-positions($tiling-x: 0px,
                                      $tiling-y: 0px,
                                      $tile-dx: 568px,
                                      $tile-dy: 0px,
                                      $line-dy: 568px,
                                      $tiles-per-line: 3, $lines: 2);
```

<p data-height="265" data-theme-id="dark" data-slug-hash="LgjBxY" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with Sass (grooves)" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/LgjBxY/">Tiled background with Sass (grooves)</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

We will also use this very same function to remake the shading on the wood. The original SVG file consists of two symmetric shades with copies offseted horizontally by a small margin, but the file we are using now has only those two symmetric shades. We will need to make three more copies for each side and change their sizes and positions to simulate the same effect. Since doing that would also leave empty gaps where there shouldn't be, we will need to increase their overall sizes too. This time however, two instances of `tiling-positions()` are needed; one for each side.

We first increase the number of image to `16` (a grid of `4 x 2` for each side).

```
background-image: tiling-images($path-to-shades, 16);
```

Then set the `background-position`s like these:

```
background-position: tiling-positions($tiling-x: 12%,
                                      $tiling-y: 0px,
                                      $tile-dx: 14%,
                                      $tile-dy: 0px,
                                      $line-dy: $shade-height,
                                      $tiles-per-lilne: 4, $lines: 2),
                     tiling-positions($tiling-x: 88%,
                                      $tiling-y: 0px,
                                      $tile-dx: -14%,
                                      $tile-dy: 0px,
                                      $line-dy: $shade-height,
                                      $tiles-per-lilne: 4, $lines: 2);
```

<p data-height="265" data-theme-id="dark" data-slug-hash="LgeKqa" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with Sass (shades)" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/LgeKqa/">Tiled background with Sass (shades)</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

The **chains**, however, will present us a new problem. Both pair of chains (the ones on the top and on the bottom) are made by a single SVG file with links seen sideways and two other seen from the front.

![chainLink.svg][chain link]

The purpose of such an arragment is to create two lines of images, one for the links seen from the side and another for the ones seen from the front, and set their `background-size` to be double than that of its container so as to show only one view of the links at a time while using just a single SVG file.

```
background-image: tiling-images($path-to-chain-link, 2);
background-position: 0 calc(100% + 3px), 100% calc(100% + 43px);
background-size: 200% $chain-link-height;
```

The first link of each chain is the one seeen from the side. Since this is the first link in the SVG file from left to right, we set the fist `background-position` to be on the left, hence `x = 0`. On the other side, the second image is located on the right side of the file, so we set the position of the second image to be `100%`, that is `x = 100%`.

As for their `y` coordinates, the tip of the side links should barely touch the bottom of the container so we add `3px` to `100%` to remove the empty gap in the SVG file. The same goes for the links seen from the front, only this time the required amount is `-43px`.

That being said, the links don't quite align yet. Their images get cut halfway through. This is because the **iterator** inside `_tiling-line-positions()` only increases by `1` and in this case, we need it to increase according to the formula `2 * $i - 1`, but alas, Sass doesn't have a feature to control a `@for` loop **iterator** directly. The answer however, is quite simple. Instead of trying to control the **iterator**, we just need to replace the `$i` in `$tile-dx * $i + $line-x` and `$tile-dy * $i + $line-y` by `$a * $i + $b` so that we get:

`$tile-dx * ($a * $i + $b) + $line-x`

and

`$tile-dy * ($a * $i + $b) + $line-y`

Where `$a` and `$b` are two new parameters to be set by the function's user and so we update both `_tiling-line-positions()` and `tiling-positions()` to include them as an option.

It's time to put this new modification to the test starting with the top chains. As explained above, two lines of images are needed for each pair of chains. The one with the links seen from the side goes first because those must be rendered in front of the others (from the user's point of view). Since the SVG of the links already includes the empty gaps between them, the vertical distance between each `background-image`, that is `$tile-dy`, should be the same as the height of the SVG itself. It must be negative too, because it goes from bottom to top. The top chains should stick to the bottom, so the position of the entire line of images, or `$line-y` should start at `100% - $chain-link-height` plus `3px` for better positioning. For the purpose of this example, we will set to `10` the number of `$tiles` per line, but you can see I used a lot more in practice just to make sure the chains fit nicely even in crazy high resolution screens.

The second line will display the images of the links seen from the front. It's position, or `$line-y` should be almost at the bottom of the container. `calc(100% + 3px - #{$chain-link-height})` to be precise. The `$tile-dy` this time should be half of the SVG's height, so we set it to `calc(#{-$chain-link-height} / 2)`. And finally, the moment we were all waiting for,  we set the multiplier `$a` to `2` and `$b` to `-1` to get the right distance between these links. Lastly, we set `$lines` to `10` as well.

Rinse and repeat for the bottom chains, removing `100%` and the minus sign in both lines' `$line-y` and `$tile-dy` values.

The final result of our hard work looks as follows.

<p data-height="265" data-theme-id="dark" data-slug-hash="MPqKBe" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with Sass (chains)" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/MPqKBe/">Tiled background with Sass (chains)</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

There is one more important function that is still missing and that's the one for writing multiple `background-size`s. The reason is simply; different `background-image`s will require different `background-size`s. For example, we will need to set the `80` images of the **shades** to `200% $shade-height` while `100` of **grooves** will need to be set at `$grooves-width` for both `width` and `height`. However, this function will proove to be very easy to create.

```scss
@function tiling-sizes($dimensions: inherit, $amount: 1) {
  $result: null;

  @for $i from 1 through $amount {
    $result: $result, $dimensions;
  }

  @return $result;
}
```

It's basically a much simpler version of `tiling-positions()` since we will not be doing any weird operations with it, just `@return`ing multiple values.

Just perfect! All we need to do now is wrap all this up with some `@mixin`s and we will be good to go.

The first `@mixin` we will create is the one for the planks.

```scss
@mixin planks-multiple-backgrounds($corner-width) {
  background-clip: padding-box;
  background-color: $color-planks-diffuse;
  background-image: tiling-images($path-to-shades, 80),
                    tiling-images($path-to-grooves, 100);
  background-position: tiling-positions($tiling-x: 12%,
                                        $tiling-y: 0px,
                                        $tile-dx: 14%,
                                        $tile-dy: 0px,
                                        $line-dy: $shade-height,
                                        $tiles-per-lilne: 4, $lines: 10),
                       tiling-positions($tiling-x: 88%,
                                        $tiling-y: 0px,
                                        $tile-dx: -14%,
                                        $tile-dy: 0px,
                                        $line-dy: $shade-height,
                                        $tiles-per-lilne: 4, $lines: 10),
                       tiling-positions($tiling-x: 0px,
                                        $tiling-y: 0px,
                                        $tile-dx: $grooves-width,
                                        $tile-dy: 0px,
                                        $line-dy: $grooves-height,
                                        $tiles-per-lilne: 10,
                                        $lines: 10);
  background-size: background-sizes(200%, $shade-height, 80),
                   background-sizes($grooves-width, $grooves-height, 100);
  background-repeat: no-repeat;
  box-sizing: border-box;
  clip-path: planks-clip($corner-width);
}
```

The only required argument is the `$corner-width` since this is what will be used to clip the `background-images` according to the type of board it is with the help of the `planks-cliip()` function we made earlier.

Notice that this time I used a lot more images than in the previous examples; `10` lines of `4` images for the shades on each side, which amounts to a total of `80` images, and `10` lines of `10` images each for the grooves. As you might have realized, we could have created a more specific function for the shades, but I preferred not to to avoid adding too many layers of abstraction and thus making it more difficult to understand how it all works.

The second `@mixin` is the one for remaking the chains.

```scss
@mixin chains-multiple-backgrounds($link-height, $chains-y, $chains-width) {
  @include center-horizontally($sign-holes-width);
  background-image: tiling-images(url($path-to-chain-link), 20);
  background-position: tiling-positions($tiling-x: 0px,
                                        $tiling-y: $chains-y,
                                        $tile-dx: 0px,
                                        $tile-dy: $link-height,                                        
                                        $tiles-per-line: 10),
                       tiling-positions($tiling-x: 100%,
                                        $tiling-y: calc(#{strip-calc($chains-y)} + #{$link-height}),
                                        $tile-dx: 0px,
                                        $tile-dy: calc(#{$link-height} / 2),
                                        $a: 2,
                                        $b: -1,
                                        $tiles-per-line: 10);
  background-size: tiling-sizes(200% $chain-link-height);
  background-repeat: no-repeat;
  height: 100%;
}
```

The parameters of this `@mixin` are as fallow:

- `$link-height`: the height of the **link**.
- `$chains-y`: the `y` coordinate of the **chains**.
- `$chains-width`: the width of the **image** containing the chains graphic (equal to the width of the are where the holes in the board are, that is, `$bulletin-holes-width` or `$sign-holes-width`).

With the `@mixin`s completed and ready to roll we can finally fix the issue of alpha transparency not being rendered properly in Safari 11.

For **bulletin** type board (inside the `&--type-bulletin` selector):

```scss
@at-root .jpeg2000.peerconnection & {
  background-image: none; // Just make sure no background is displayed.

  div {
    position: absolute;
  }

  & div:nth-child(12) {@include chains-multiple-backgrounds(-$chain-link-height, calc(#{strip-calc($sign-top-chains-height)} - 63px), $bulletin-holes-width);}
  & div:nth-child(1) {@include planks-multiple-backgrounds($bulletin-corner-width);}
}
```

For the **sign** one (inside the `&--type-sign` selector):

```scss
@at-root .jpeg2000.peerconnection & { // Change back to .jpeg2000 after testing.
  background-image: none; // Just make sure no background is displayed.

  div {
    position: absolute;
  }

  & div:nth-child(13) {@include chains-multiple-backgrounds(-$chain-link-height, calc(#{strip-calc($sign-top-chains-height)} - 63px), $sign-holes-width);}
  & div:nth-child(12) {@include chains-multiple-backgrounds($chain-link-height, calc(#{strip-calc($sign-bottom-chains-div-position-top)} + 39px), $sign-holes-width);}
  & div:nth-child(11) {@include corners($path-to-sign-corners, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(10) {@include horizontal-sides($path-to-sign-horizontal-sides, $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(9) {@include vertical-sides($path-to-vertical-sides, $sign-vertical-height, $sign-board-height, $sign-side-position-left, 2);}
  & div:nth-child(8) {@include holes($path-to-sign-holes, $sign-holes-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(7) {@include horizontal-sides($path-to-horizontal-sides-depth, $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(6) {@include corners($path-to-sign-corners-depth, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(5) {@include horizontal-sides($path-to-horizontal-sides-shadow, $sign-horizontal-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(4) {@include vertical-sides($path-to-vertical-sides-shadow, $sign-vertical-height, $sign-board-height, $sign-side-position-left, 2);}
  & div:nth-child(3) {@include corners($path-to-sign-corners-shadow, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(2) {@include holes($path-to-sign-holes-shadow, $sign-holes-shadow-width, $sign-board-height, $sign-board-position-top);}
  & div:nth-child(1) {@include planks-multiple-backgrounds($sign-corner-width);}
}
```

With this last step we have covered all the issues worth fixing. All we need to do is to display a message prompting the user to update their browser or device if they don't support one of the required features. Since we might be doing this more than one time, a `@mixim` will come really in handy.

```scss
@mixin full-page-msg($msg) {
  content: $msg;
  font-family: MedievalSharp;
  font-size: 2em;
  margin-top: -0.5em;
  position: absolute;
  text-align: center;
  top: 50%;
  color: black;
  width: 100%;
}
```

The required features in case are CSS **calc()**, **gradients** and **generated content**. If one of them is not supported, then the message will be appended to the HTML document itself so we use `@at-root` directive to keep this rule tidely packed inside `.board`. Of course, the board should be hidden completely.

Also, Firefox 15 and older don't display SVG elements that use viewbox attribute, so in this case we have to use a combination of two Modernizr detects; **sandbox** and **cssmozoutlineradius** (for targeting Firefox specifically) to decide whether to display or not the board. Note that by doing so, Firefox 16 will also be considered as an unsupported browser for our convinience.

```scss
@at-root .no-csscalc,
         .no-cssgradients,
         /* IE < 10 dont't support innerHTML on html elements. */
         .no-generatedcontent,
         .cssmozoutlineradius.no-sandbox
        {
  /* Remember, IE8 only supports the single-colon CSS 2.1 syntax (i.e. :pseudo-class). This is fixed width Autoprefixer. */
  ::after {
    @include full-page-msg($browser-is-too-old);
  }

  .board {
    display: none;
  }
}
```

Finally! With this we have succesfully made sure the site reacts properly to the user's browser capabilities by **degrading gracefully** if a feature is not supported or displaying a friendly message if not even that works. There is probably a lot more room for improvement but now we have a solid foundation to work upon. One of those improvements is something easy to overlook when working with Sass. I'm talking about **error handling**. We will touch on this topic in the [next post][ch-18] and with it give a proper closure to our adventure packed journey.



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
[`index.processed.css`]: https://codepen.io/andresangelini/project/editor/Aarxxz
[shapes]: https://css-tricks.com/the-shapes-of-css/
[if]: https://sass-lang.com/documentation/file.SASS_REFERENCE.html#if
[for]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#for
[unitless]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[error]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#error
[chain link]: https://cdn.rawgit.com/andresangelini/5969d4f442bb18ec3b81db61ab4202fe/raw/6a51402d5877ccfec47937952937af0d39aa7ddc/chainLink.svg "Chain Link SVG"
