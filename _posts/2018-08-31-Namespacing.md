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

### Namespacing

Now that we learned about [the existence of `defs` and how to take advantage of them properly][ch-6], we should start thinking about using some kind of naming system for our elements IDs, so that we can clearly recognize what is what and thus avoid any potencial name collision. So far, we have three types of elements: basic shapes (the actual base graphics, such as `<path>`), their modifications (i.e.: our reused path made to stretch) and their implementations (when we `<use>` something outside `<defs>` ). This is where [namespaces] really come in handy. Basically, we are going to prepend an abbreviated version of its category to each element name. I'll go ahead and add some usefull categories and their corresponding namespaces into a table for future reference.

| Name          | Namespace | Definition  |
| ------------- |:---------:| -----------:|
| basic shape   | bs        | A shape defined only by its structure without any modification applied and meant to be reused.|
| component     | c         | A reusable part made up from one or more basic shapes or even other components with modifications applied.      |
| pattern       | p         | A design used to fill a basic shape or component.|
| clip path     | cp        | A path used to define the visible area of a basic shape or component.
| layer         | l         | The final result meant to be used as a background image.|

Let's use the top part of the frame again as an example to see how this would look like.

```xml
<svg version="1.1" xmlns="http://www.w3.org/2000/svg">
  <!-- Reusable graphics or templates. -->
  <defs>
    <!-- BASIC SHAPES -->
    <path id='bs-side' d='m 0,0 c 301.53,5.7865 603.05,5.7864 904.58,0 l 0,11.9 C 603.1,17.691 301.57,17.692 0,11.9 Z'/>

    <!-- COMPONENTS -->
    <symbol id='c-horizontal-side' width='100%' height='16.25' viewBox='0 0 905 16.25' preserveAspectRatio='none'>
      <use xlink:href='#side' fill='#442d18'>
    </symbol>
  </defs>

  <!-- Use the graphic. -->
  <use xlink:href='#c-horizontal-side'>

</svg>
```

Pretty neat, right? But we will see the true usefullness of namespaces once we have all the pieces of the board altogether. Next, we will learn [how to style an SVG like a pro][ch-8].



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
