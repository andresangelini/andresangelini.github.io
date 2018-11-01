## An otherwise mess of declarations

As briefly mentioned at the end of the last post, it would be an unimaginably tedious task, to say the least, if you had to remake an SVG pattern using multilpe `background-image`s in vanilla CSS. Just to give you an idea of how incredibly messy this would be, have a look at the line `303` of [`index.processed.css`] where the `background-image`s for the `planks` are set. If you are thinking a **mixin** could solve the problem, think again. **Mixins** can only return **declarations** and we don't want to create multiple declarations for `background-image` because we would only be overriding the previous one instead of setting multiple ones. To understand better what we need to do, let's recap how each SVG pattern is made.

The planks have actually three separate layers: a plain **background color**, the **grooves** of the wood and their **shades**. Each of them behave differently but all of them must be **clipped** so that they don't overflow through the wooden frame. Basically, as the board expands and contracts, each layer should:

- **background color**: fill all available space.
- **grooves**: repeat both horizontally and vertically.
- **shades**: have a simetric copy on each side of the board, both repeating horizontally and vertically.

Let's start with what seems easier; the **background color**.

## Clipping in CSS

Safari 11 might not support SVG patterns with alpha but it does have partial support for CSS `clip-path` if you use it along with CSS [shapes]. These last ones are a relatively new feature that allow us to draw simple figures in CSS. You have your basic shapes such as `line()`, `rectangle()` and `circle()`. To create a figure using both you only need to set a `clip-path` **property** with any of these shapes as a **value** to a `div`. For example:

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

All the usual units are allowed which is exactly what we need in order for this clip-path to be flexible. However, the boards differ not only in the size of their corners but the sign type also has holes in the bottom, so it would be wise to write a function that returns a `polygon()` depending on which board it is. To do this, we will need to use another great feature of Sass; the built-in [`if()`][if] function (not to be confused with the [`@if`][if] directive).

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

The result is a function does everything for us by simply changing its argument to the correspondant `$corner-width`.

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

First off, let's make a simple function for creating multiple `background-image`s. It should have two parameters; the `$path` URL to the image and the `$amount` of copies we want. A [`@for`][for] loop will do the trick here.

```scss
// Returns multiple backgroun images.
@function bkg-imgs($path, $amount) {
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

In other words, the positions of the images can be described by the general ecuation of a straight line `$img-dx * $i + $line-x` for `x` axis and `$img-dy * $i + $line-y` for `y`, where:

- `$img-dx` and `$img-dy`are the horizontal and vertical distances between each `background-image`.
- `$i` is the `@for` loop **iterator** used to increase the distance of the **line** from the container's top left corner.
- `$line-x` and `$line-y` are the coordinates of the **line** of `background-image`'s top left corner.

With this, we will create a new function called `bkg-pos-line` which will loop `$imgs` number of times to concatenate each new `$result` to the previous one as `$i` increases and returns multiple `background-postion`'s in a **line** pattern.

```scss
@function bkg-pos-line($line-x: 0px, $line-y: 0px, $img-dx: 0px, $img-dy: 0px, $imgs: 1) {
  $result: null;

  @for $i from 0 to $imgs {
    $result: $result, calc((#{$img-dx} * #{$i}) + #{$line-x}) +
                      " " +
                      calc((#{$img-dy} * #{$i}) + #{$line-y});
  }

  @return $result;
}
```

Now we must the same logic to obtain a **grid** using multiple **lines** of images instead of just images. This time, the ecuations change to `$line-dx * $i + $grid-x` and `$line-dy * $i + $grid-y` for each `$line-x` and `$line-y` where:

- `$line-dx` and `$line-dy` are the horizontal and vertical distances between each line pattern.
- `$i` is the `@for` loop **iterator** used to increase the distance of the **grid** from the container's top left corner.
- `$grid-x` and `$grid-y` are the coordinates of the grid pattern's top left corner.

The idea behind this is to create a **grid** out of **lines** using the `bkg-pos-line()` function we just created and set the coordinates for each **line** as the **iteraror** increases so we don't have to write the same function over and over again to get more than one line of `background-images`:

```
background-positions: bkg-pos-line($line-x: 0px,
                                   $line-y: 0px,
                                   $img-dx: 568px,
                                   $img-dy: 0px,
                                   $imgs: 3),
                      bkg-pos-line($line-x: 0px,
                                   $line-y: 568px,
                                   $img-dx: 568px,
                                   $img-dy: 0px,
                                   $imgs: 3);
```

This positions 3 images in a row next to each other without any gap in between and another 3 right below them also in a line. However, our new function called `bkg-pos-grid` will make writing this much easier and it looks like this:

```scss
@function bkg-pos-grid($grid-x: 0px,
                       $grid-y: 0px,
                       $img-dx: 0px,
                       $img-dy: 0px,
                       $line-dx: 0px,
                       $line-dy: 0px,
                       $imgs: 1,
                       $lines: 1) {
  $result: null;

  @for $i from 0 to $lines {
    $result: $result, bkg-pos-line(calc((#{$line-dx} * #{$i}) + #{$grid-x}), calc((#{$line-dy} * #{$i}) + #{$grid-y}), $img-dx, $img-dy, $imgs);
  }

  @return $result;
}
```

And we can set the `background-positions` for the `grooves` like this:

```
background-position: bkg-pos-grid($grid-x: 0px,
                                    $grid-y: 0px,
                                    $img-dx: 568px,
                                    $img-dy: 0px,
                                    $line-dy: 568px,
                                    $imgs: 3, $lines: 2);
```

<p data-height="265" data-theme-id="dark" data-slug-hash="LgjBxY" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with Sass (grooves)" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/LgjBxY/">Tiled background with Sass (grooves)</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

We will also use this very same function to remake the shading on the wood. The original SVG file consists of two symmetric shades with copies offseted horizontally by small margin, but the file we are using now has only those two symmetric shades. We will need to make three more copies for each side and change their sizes and positions to simulate the same effect. Since doing that would also leave empty gaps where there shouldn't be, we will need to increase their overall sizes too. This time however, two instances of `bkg-pos-grid()` function are needed; one for each side.

We first increase the number of image to `16` (a grid of `4 x 2` for each side).

```
background-image: background-images($path-to-shades, 16);
```

Then set the `background-position`s like these:

```
background-position: bkg-pos-grid($grid-x: 12%,
                                  $grid-y: 0px,
                                  $img-dx: 14%,
                                  $img-dy: 0px,
                                  $line-dy: $shade-height,
                                  $imgs: 4, $lines: 2),
                     bkg-pos-grid($grid-x: 88%,
                                  $grid-y: 0px,
                                  $img-dx: -14%,
                                  $img-dy: 0px,
                                  $line-dy: $shade-height,
                                  $imgs: 4, $lines: 2);
```

<p data-height="265" data-theme-id="dark" data-slug-hash="LgeKqa" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with Sass (shades)" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/LgeKqa/">Tiled background with Sass (shades)</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

The **chains**, however, will present us a new problem. Both pair of chains (the ones on the top and on the bottom) are made by a single SVG file with links seen sideways and two other seen from the front.

![chainLink.svg][chain link]

The purpose of such an arragment is to create two lines of images, one for the links seen from the side and another for the ones seen from the front, and set their `background-size` to be double than that of its container so as to show only one view of the links at a time while using just a single SVG file.

```
background-image: bkg-img($path-to-chain-link, 2);
background-position: 0 calc(100% + 3px), 100% calc(100% + 43px);
background-size: 200% $chain-link-height;
```

The first link of each chain is the one seeen from the side. Since this is the first link in the SVG file from left to right, we set the fist `background-position` to be on the left, hence `x = 0`. On the other side, the second image is located on the right side of the file, so we set the position of the second image to be `100%`, that is `x = 100%`.

As for their `y` coordinates, the tip of the side links should barely touch the bottom of the container so we add `3px` to `100%` to remove the empty gap in the SVG file. The same goes for the links seen from the fron, only this time the required amount is `-43px`.

That being said, the links don't quite align yet. Their images get cut halfway through. This is because the **iterator** inside the **bkg-pos-line** function only increases by `1` and in this case, we need it to increase according to the formula `2 * $i - 1`, but alas, Sass doesn't have a feature to control a `@for` loop **iterator** directly. The answer however, is quite simple. Instead of trying to control the **iterator**, we just need to replace the `$i` in `$img-dx * $i + $line-x` and `$img-dy * $i + $line-y` by `$a * $i + $b` so that we get:

`$img-dx * ($a * $i + $b) + $line-x`

and

`$img-dy * ($a * $i + $b) + $line-y`

Where `$a` and `$b` are two new parameters to be set by the function's user and so we update both `bkg-pos-line()` and `bkg-pos-grid()` functions to accept them.

It's time to put this new modification to the test starting with the top chains. As explained above, two lines of images are needed for each pair of chains. The one with the links seen from the side goes first because those must be rendered in front of the others (from the user's point of view). Since the SVG of the links already includes the empty gaps between them, the vertical distance between each `background-image`, that is `$img-dy`, should be the same as the height of the SVG itself. It must be negative too, because it goes from bottom to top. The top chains should stick to the bottom, so the position of the entire line of images, or `$line-y` should start at `100% - $chain-link-height` plus `3px` for better positioning. For the purpose of this example, we will set to `10` the number of `$imgs` per line, but you can see I used a lot more in practice just to make sure the chains fit nicely even in crazy high resolution screens.

The second line will display the images of the links seen from the front. It's position, or `$line-y` should be almost at the bottom of the container. `calc(100% + 3px - #{$chain-link-height})` to be precise. The `$img-dy` this time should be half of the SVG's height, so we set it to `calc(#{-$chain-link-height} / 2)`. And finally, the moment we were all waiting for,  we set the multiplier `$a` to `2` and `$b` to `-1` to get the right distance between these links. Lastly, we set `$imgs` to `10` as well.

Rinse and repeat for the bottom chains, removing `100%` and the minus sign in both lines' `$line-y` and `$img-dy` values.

The final result of our hard work looks as follows.

<p data-height="265" data-theme-id="dark" data-slug-hash="MPqKBe" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Tiled background with Sass (chains)" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/MPqKBe/">Tiled background with Sass (chains)</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

Just perfect! All we need to do now is wrap all this up with some `@mixin`s and we will be good to go.

The first `@mixin` we will create is the one for the planks.

```scss
@mixin planks-multiple-backgrounds($corner-width) {
  background-clip: padding-box;
  background-color: $color-planks-diffuse;
  background-image: bkg-imgs($path-to-shades, 80),
                    bkg-imgs($path-to-grooves, 100);
  background-position: bkg-pos-grid($grid-x: 12%,
                                    $grid-y: 0px,
                                    $img-dx: 14%,
                                    $img-dy: 0px,
                                    $line-dy: $shade-height,
                                    $imgs: 4, $lines: 10),
                       bkg-pos-grid($grid-x: 88%,
                                    $grid-y: 0px,
                                    $img-dx: -14%,
                                    $img-dy: 0px,
                                    $line-dy: $shade-height,
                                    $imgs: 4, $lines: 10),
                       bkg-pos-grid($grid-x: 0px,
                                    $grid-y: 0px,
                                    $img-dx: $grooves-width,
                                    $img-dy: 0px,
                                    $line-dy: $grooves-height,
                                    $imgs: 10,
                                    $lines: 10);
  background-size: background-sizes(200%, $shade-height, 80),
                   background-sizes($grooves-width, $grooves-height, 100);
  background-repeat: no-repeat;
  box-sizing: border-box;
  clip-path: planks-clip($corner-width);
}
```

The only required argument is the `$corner-width` since this is what will be used to clip the `background-images` according to the type of board it is with the help of the `planks-cliip()` function we made earlier.

Notice that this time I used a lot more images than in the previous examples; `10` lines of `4` images for the shades on each side, which amounts to a total of `80` images, and `10` lines of `10` images each for the grooves. As you might have realized, we could have created a more specific function for the shades, but I preferred not to avoid adding too many layers of abstraction and thus making it more difficult to understand how it all works.

The second `@mixin` is the one for remaking the chains.

```scss
@mixin chains-multiple-backgrounds($chain-link-height, $offset-y) {
  background-image: bkg-imgs($path-to-chain-link, 20);
  background-position: bkg-pos-line($line-x: 0px,
                                    $line-y: calc(100% + #{$offset-y}),
                                    $img-dx: 0px,
                                    $img-dy: -$chain-link-height,
                                    $imgs: 10),
                       bkg-pos-line($line-x: 100%,
                                    $line-y: calc(100% + #{$offset-y} - #{$chain-link-height}),
                                    $img-dx: 0px,
                                    $img-dy: calc(#{-$chain-link-height} / 2),
                                    $a: 2,
                                    $b: -1,
                                    $imgs: 10);
  background-size: 200% $chain-link-height;
  background-repeat: no-repeat;
}
```

We will leave `$chain-link-height` and `$offset-y` as reuired parameters in case we make any modification to the graphics in the future.

With the `@mixin`s completed and ready to roll we can finally the issue of alpha transparency not being rendered properly in Safari 11.

For **bulletin** type board (inside the `&--type-bulletin` selector):

```scss
@at-root .jpeg2000.peerconnection & {
  & div:nth-child(12) {@include chains-multiple-backgrounds($chain-link-height, 3px);}     
  & div:nth-child(1) {@include planks-multiple-backgrounds($bulletin-corner-width);}
}
```

For the **sign** one (inside the `&--type-sign` selector):

```scss
@at-root .jpeg2000.peerconnection & { // Change back to .jpeg2000 after testing.

  div {
    background: none; // Just make sure no background is displayed.
  }

  & div:nth-child(13) {@include chains-multiple-backgrounds($chain-link-height, 3px);}
  & div:nth-child(12) {@include chains-multiple-backgrounds($y: -3px, $offset-y: $chain-link-height);}
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

With this last step he have covered all the issues worth fixing. All we need to do is to display a message prompting the user to update their browser or device if they don't support one of the required features. Since we might be doing this more than one time, a `@mixim` will come really in handy.

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

The required features in case are CSS **calc()**, **gradients** and **generated content**. If one of them is not supported then the message itself will be appended to the HTML document itself so we use `@at-root` directive to keep this rule tidely packed inside `.board`. Of course, the board should be hidden completely.

Also, Firefox 15 and older don't display SVG elements that use viewbox attribute, so in this case we have to use a combination of two Modernizr detects; **sandbox** and **cssmozoutlineradius** (for targeting Firefox specifically) to decide whether to display or not the board. Note that as a compromise Firefox 16 which does display the SVG properly but it will be skipped for convenience.

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

Finally! With this we have made sure the site will try to **degrade gracefully** if a feature is not supported or at display a friendly message if not even the fallback method works. There is probably a lot more room for improvement but now we have a solid foundation to work upon. One of those improvements is something that is easy to forget when working with Sass. I'm talking about **error handling**. We touch on this topic in the next post and with it give a proper closure to our adventure packed journey.




[`index.processed.css`]: https://codepen.io/andresangelini/project/editor/Aarxxz
[shapes]: https://css-tricks.com/the-shapes-of-css/
[if]: https://sass-lang.com/documentation/file.SASS_REFERENCE.html#if
[for]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#for
[unitless]: http://sass-lang.com/documentation/Sass/Script/Functions.html#unitless-instance_method
[error]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#error
[chain link]: https://cdn.rawgit.com/andresangelini/5969d4f442bb18ec3b81db61ab4202fe/raw/6a51402d5877ccfec47937952937af0d39aa7ddc/chainLink.svg "Chain Link SVG"
