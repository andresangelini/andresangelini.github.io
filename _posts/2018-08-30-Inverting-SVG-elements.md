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

### Inverting SVG elements

In this brief post we will continue [experimenting with the SVG's capabilities][ch-2]  and see if we can invert an element. Sadly for us, there is no such thing as an "invert" transformation, but we do have an `scale()` parameter that we can use to achieve the same effect. `scale()` only allows ranges from `1` to `-1`, where `1` will do nothing on that axis, `0` will make it infinitely thin, to the point of dissapearing completely, and `-1` will actually flip it. There is one caveat, though. The flipping depends on where the element's axis is, which usually is on its top left corner if such element was not created in a vector graphic software, as we talked about earlier. This means that in most cases, when you apply a `scale()`, the element will also be translated, so we have to make the same type of correction we do when rotating, but this time using the element's `x` and `y` attributes becuase these ones, contrary to `translate()`, allows us to use values in percentages.

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
  <!-- Make graphic stretchable with <symbol>. -->
  <symbol id='vertical-side' width='16.25px' height='100%' viewBox='0 0 16.25 905' preserveAspectRatio='none'>
    <!-- Create and rotate graphic. -->
    <path transform='rotate(-90) translate(-905 0)' fill='#442d18' d='m 0,0 c 301.53,5.7865 603.05,5.7864 904.58,0 l 0,11.9 C 603.1,17.691 301.57,17.692 0,11.9 Z'/>
  </symbol>

  <!-- Use rotated graphic for the left side and an inverted copy for the righ side. -->
  <use xlink:href='#vertical-side'/>
  <use xlink:href='#vertical-side' x='-100%' transform='scale(-1 1)'/>
</svg>
```
Using the rotated side as an example, here we apply a `scale()` transform and set its `x` to `-100%` to move it back to its right location. The final result should look like this:

<p data-height="265" data-theme-id="0" data-slug-hash="ZaRwww" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="Inverting an SVG element" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/ZaRwww/">Inverting an SVG element</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Next up, it's [rotation][ch-4] time!



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
