## An otherwise mess of declarations

As briefly mentioned at the end of the last post, it would be an unimaginably tedious task, to say the least, if you had to remake an SVG pattern using multilpe `background-image`s in vanilla CSS. Just to give you an idea of how incredibly messy this would be, have a look at the line `303` of [`index.processed.css`] where the `background-image`s for the `planks` are set. If you are thinking a **mixin** could solve the problem, think again. **Mixins** can only return **declarations** and we don't want to create multiple declarations for `background-image` because we would only be overriding the previous one instead of setting multiple ones. To understand better what we need to do, let's recap how each SVG pattern is made.

The planks are have actually three separate layers: a plain **background color**, the **grooves** of the wood and their **shades**. Each of them behave differently but all of them must be **clipped** so that they don't overflow through the wooden frame. Basically, as the board expands and contracts, each layer should:

- **background color**: fill all available space.
- **grooves**: repeat both horizontally and vertically.
- **shades**: have a simetric copy on each side of the board, both repeating horizontally and vertically.

Let's start with what seems easier; the **background color**.

## Clipping in CSS

Safari 11 might not support SVG patterns with alpha but it does have partial support for CSS `clip-path` if you use it along with CSS [shapes]. These last ones are a relatively new feature that allow us to draw simple figures in CSS. You have your basic shapes such as `line`, `rectangle` and `circle`. To create a figure using both you only need to set a `clip-path` **property** with any of these shapes as a **value** to a `div`. For example:

```css
clip-path: circle(50px);
```

What we are actually doing, in fact, is creating a regular squared `div` and clipping it with the shape we want it to have, as shown below.

<p data-height="265" data-theme-id="0" data-slug-hash="PypmBO" data-default-tab="css,result" data-user="andresangelini" data-pen-title="Simple CSS shape" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/PypmBO/">Simple CSS shape</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

But you can also build more complex figures using `polygon`, which is what we are going to use to reproduce our clipped SVG pattern.

<p data-height="265" data-theme-id="0" data-slug-hash="vpYbPP" data-default-tab="result" data-user="andresangelini" data-pen-title="Clipping SVG elements with complex shapes" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/vpYbPP/">Clipping SVG elements with complex shapes</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

To draw a shape using `polygon`, we once again set it as the **value** of the `clip-path` **property**. Each point or vertex is written as a pair of coordinates separated by a **comma** starting from the **top left corner** and going **clockwise**.

```css
cip-path: polygon(x-1 y-1, x-2 y-2, x-3 y-3, ...);
```

All the usual units are allowed which is exactly what we need in order for this clip-path to be flexible.


[`index.processed.css`]: https://codepen.io/andresangelini/project/editor/Aarxxz
[shapes]: https://css-tricks.com/the-shapes-of-css/
