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

### Clipping SVG elements

Looking carefully at the [top side of the frame] of the medieval board we were rotating in the [last post][ch-4], you'll notice that if left as it is, it will visible right through the holes of the top chains. Since we want to reuse this piece for all four sides of the frame, we would have to find some way to "cut" a hole right where the holes are without modifying the other sides. This is where the `clip-path` attribute comes into play. Its only parameter, `url()`, allows us to reference to **a path or any basic shape** to define **the visible area** of our element. This might feel counter intuitive at first since most image manipulation software out there have us accostumed to the opposite. Also, note that `<symbol>` elements cannot be used as `clip-path`. This is quite sad because their ability of being stretcheable through the use of `viewBox` and `preserveAspectRatio` would really come in handy. Fortunately for us, there is another way this can be dealt with. Let's try to remove the center part of the frame's top side where the entire title area should be. Remember though, what we have to define is the visible area, which means we need to use three separated pieces: one to cover just a little bit of top side in its entire width, and one for each side.

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">

  <!-- Clip path -->
  <clipPath id='top-side-clip-path'>
    <rect x='0' y='0' width='100%' height='7'/>
    <rect x='-90%' y='0' width='100%' height='100%' transform='translate(3.16 0)'/>
    <rect x='90%' y='0' width='100%' height='100%' transform='translate(-3.16 0)'/>
  </clipPath>

  <!-- Reusable graphic or template. -->
  <symbol id='horizontal-side' width='100%' height='16.25' viewBox='0 0 905 16.25' preserveAspectRatio='none'>
    <path d='m 0,0 c 301.53,5.7865 603.05,5.7864 904.58,0 l 0,11.9 C 603.1,17.691 301.57,17.692 0,11.9 Z' fill='#442d18'/>
  </symbol>

  <!-- Use the graphic. -->
  <use xlink:href='#horizontal-side' clip-path='url(#top-side-clip-path)'/>
</svg>
```
The end result should look something like this:

<p data-height="265" data-theme-id="0" data-slug-hash="ooqder" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="clip-path on a stretchable SVG element" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/ooqder/">clip-path on a stretchable SVG element</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

That's it for now. In the [next post][ch-6], we will be taking a short detour to talk about how to orginze our SVG elements to work more efficiently.



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
