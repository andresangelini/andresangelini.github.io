This post is just one part of a larger story about an alchemist's personal quest for [Making a Responsive Medieval Board With SVG Stacks][ch-1]. You may read the chapters in any order you want but I would otherwise suggest you to do it in the proper order to get the full context of the project.

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

### Rotating SVG elements

One of the best features of SVGs is their ability to transform reusable elements. This is easily done by applying a `transform` attribute which has many parameters that can be chained together by leaving a space between them. In this post we will focus on the `rotate()` and `translate()` parameters for now. While `rotate()` only accepts a single unitless value representing degrees, `translate()` takes a unitless value pair (positive or negative), one for each axis. Let's use a copy of the top side of the frame we created on the [previous post][ch-3] to see how this works.

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
  <!-- Make graphic stretchable with <symbol>. -->
  <symbol id='vertical-side' width='16.25px' height='100%' viewBox='0 0 16.25 905' preserveAspectRatio='none'>
    <!-- Create and rotate graphic. -->
    <path transform='rotate(-90) translate(-905 0)' d='m 0,0 c 301.53,5.7865 603.05,5.7864 904.58,0 l 0,11.9 C 603.1,17.691 301.57,17.692 0,11.9 Z'/>
  </symbol>

  <!-- Use the rotated graphic. -->
  <use href:xlink='#vertical-side'>
</svg>
```

Applying the rotation might result surprisingly difficult if you don't know where the element's origin actually is, so locating it should be our first priority. Usually it's on the top left corner of the element, but it might be in a different location if you created the element in a vector graphic application. In this case, the easiest way to modify its position is copying the `d` attribute of the element, creating a path in the application and pasting the attribute into this path in the application's xml editor (or any text editor). Then, you adjust the size of SVG to the element's size and go to the xml editor once again to copy the `d` once more. Its new value will have the element's origin in the top left corner as expected. The reason for all this trouble is because the origin's location of an element is relative to the top left corner of the SVG. For example, if you have an element smaller than the SVG and is not located on the top left corner, then this offset will be passed through the `d` attribute which will make things like rotating and scaling hard to understand. Of course we could in theory modify the `d` attribute manually, but with shapes that have a lot of points this turns up to be practically impossible for all intents and purposes. That's why we use a graphical application to have all of this done for us on the fly.

Even when the element's origin is on the top left corner, it might be somehow difficutl to get our heads around how rotating works, so I'll explain step by step what I did here.

1. First, we create our reusable shape as always.

2. Then - and this is very important - we create a `<symbol>` with the dimensions we expect the element to have after the rotation. That is, if our original element's intrinsic dimensions (because they're determined in its `d` attribute) is 905 pixels width by 16.25 pixels height, then its vertical version will be 16.25 pixels width by 905 pixels height. In other words, the original height is the vertical copy's width and the original width is now the copy's height. We also make sure to add both `viewBox` and `preserveAspectRatio` attributes so that it stretches horizontally.

3. We use a copy of our reusable shape and apply a `transform` to `rotate()` it and then `translate()` it. The reason for translating it is that, in order to leave it in the desired orientation, we have to rotate our shape -90 degrees, that is, 90 degrees counterclockwise. But by doing this, the shape will dissapear from view because it will be outside the view stablished in its containing `<symbol>`. Think of it as an actor/actress accidentally falling off the stage, if it helps. Anyways, since the original element's width is 905 px and the transformations are all applied at the same time based on the same origin, then that is the distance we need to move the copy in order to put in the right place. That's why we move it `-905px` horizontally with `translate(-905 0)`.

Since words are not always enough, here is a more visual explanation.

<p data-height="265" data-theme-id="0" data-slug-hash="gXKrVo" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="SVG element rotation" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/gXKrVo/">SVG element rotation</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Naturally, rotating is not the only thing we need to learn how to do properly with SVG elements. We also need to figure out how to clip them so that we can hide parts of our graphics to prevent them from leeking through. In our [next post][ch-5], we will be doing exactly that. Stay tunned!

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
[top side of the frame]: https://codepen.io/andresangelini/pen/gXKrVo
