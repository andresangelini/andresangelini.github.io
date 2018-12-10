This post is just one part of a larger story about an alchemist's personal quest for [Making a Responsive Medieval Board With SVG Stacks][ch-1]. You may read the chapters in any order you want but I would strongly suggest following the proper order to get the full context of the project.

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

### Improving organization with `<defs>`

[Up until now][ch-5], I've been purposely avoiding this topic and thus doing unnecessary extra work so that you can come to understannd how important it is. I'm talking about, of course, `<defs>` and how it can help us to follow the [D.R.Y.] principle.

If we look closely at our previous example, it's clear that most of the elements are basically the same but with some minor differences such as their position, size, etc. So it would be only logical to define some basic shapes and use them as templates for the rest of the parts. Well, meet `<defs>`. This element allows us to do just that; "define" a bunch of reusable elements without actually displaying them.

Let's see a simple example of how this would work:

<p data-height="265" data-theme-id="0" data-slug-hash="aVVrBW" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="Defs in SVGs" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/aVVrBW/">Defs in SVGs</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

As you can see, we "define" a basic shape and then `<use>` it inside a `<symbol>` to make it stretch, all inside the `<defs>`. Up to this point, neither the `<path>` nor the `<symbol>` are displayed. Lastly, we `<use>` the stretcheable `<symbol>` version elsewhere, outside the `<defs>`.

This is a much cleaner way of doing things but we are not quite yet. There is still one more topic we need to touch on which will in the [next post][ch-7].



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
[D.R.Y.]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
