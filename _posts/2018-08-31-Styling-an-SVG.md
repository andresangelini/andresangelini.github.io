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

### Styling an SVG

In the last two post we have been polishing things up after learning about [SVG's `defs`][ch-6] and [namespacing][ch-7]. But there is yet something else we could improve. See that "fill" attribute in our `<use>` element? We have a lot of them sharing the same color among many parts. Wouldn't it be wise to define our colors once [so we don't have to repeat ourselves](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). How could we do that, you ask? Easy. Luckily for us, SVGs have a `<style>` element that cane be used to style any element with using good old CSS like this:

```xml
<style>
        .frame--color-diffuse {fill: #442d18;}
        .frame--color-specular {fill: #76522e;}
        .frame--color-depth {fill: #2b1c0f;}
        .frame--color-shadow {fill: #68300e;}
        .plaque--color-diffuse {fill: #bfa162;}
        .plaque--color-specular {fill: #d7bb7d;}
        .plaque--color-depth {fill: #5f1802;}
        .plaque--color-shadow {fill: #2b1c0f;}
        .nail--color-diffuse {fill: #553d30;}
        .nail--color-specular {fill: #968378;}
        .nail--color-depth {fill: #372318;}
        .planks--color-diffuse {fill: #a2703f;}
        .planks--color-shade {fill: #673110;}
        .planks--color-shadow {stroke: #5f2301;}
        .planks--color-specular {stroke: #b98f65;}
        .link--color-diffuse {fill: #433d18;}
        .link--color-specular {fill: #746b2e;}
        .link--color-shadow {fill: #2a260f;}
</style>
```

The pattern you see in my class names is using the [B.E.M.] methodology. As such, their structure is `[block]-[modifier]` where I namespaced the `[modifier]` for easier reading.

This one was more of a brief addendum to the previous two posts but we have a lot more coming up in the [next one][ch-9] so be sure to stick around!



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
[B.E.M.]: http://getbem.com/introduction/
