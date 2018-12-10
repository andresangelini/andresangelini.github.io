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

### Making an SVG stretch with 'viewBox' and 'preserveAspectRatio' attributes

In the [previous post][ch-1] we analized the board's structure and divided it in several pieces according to desired behavior. Out of all the problems each of them entails, making an SVG stretch seems to be the easiest, so let's go with that first. To find a solution to this, we need to dive deep into the inner workings of SVGs, in particulary, their [Coordinate Systems and Transformations](https://www.w3.org/TR/SVG/coords.html). As I mentioned back at the begining, Sara Soueidan has an [amazing  article](https://www.sarasoueidan.com/blog/svg-coordinate-systems/) on the topic.

After reading through both materials it becomes clear that the two keys of the puzzle here are the ['viewBox'](https://www.w3.org/TR/SVG/coords.html#ViewBoxAttribute) and  ['preserveAspectRatio'](https://www.w3.org/TR/SVG/coords.html#PreserveAspectRatioAttribute) attributes. The first one, which is required for the second to work, lets you stablish the content's position and dimensions and thus the aspect ratio of `<svg>` and `<symbol>` elements and ignore their intrinsic values. The second has two parameters, `<align>` and `<meetOrSlice>`. Basically, these two dictate how the SVG's content will behave when it changes size. `<align>` has many possible options to choose from to align the content while `<meetOrSlice>` has only two: `'meet'` or `'slice'`. `'meet'` will try to fit the content in its container and `'slice'` will slice it if it doesn't fit the container. However, both of them will be ignored if `<align>` is set to `'none'`. And this is exactly what we want because it will tell the content to stretch, instead of scaling while preserving its aspect ratio.

<p data-height="265" data-theme-id="0" data-slug-hash="bYrBqe" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="stretcheable SVG" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/bYrBqe/">stretcheable SVG</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### Making SVG elements stretch using `<symbol>` and `<use>` elements.

This is all fine and dandy if all we want is to have an SVG stretch. But what we're really looking for is to have elements inside an SVG stretch, and not the SVG itself. How do we go about doing that? Well, first we need to understand a little bit how SVGs are structured. Remember that I said that the `<viewBox>` attribute could only be used in `<svg>` and `<symbol>` elements? This is becuase an SVG can also be an element, in other words, it can be nested inside an SVG. However, we will focus on the `<symbol>` element and the reason why will become clear once we learn its nature and that of the SVG itself.

As you probably already know, SVGs are no more than images drawn through math, as opposed to simple bitmap images (also called "raster images"), which is what makes them resolution independent. As such, an SVG might be composed of one or more parts, such as paths, basic shapes, texts, etc., which themselves represent graphic objects. However, it might also have elements which are not graphic in nature but are rather used to define graphical template objects to be used later on. These elements are the `<symbol>` element. Basically, you nest a graphical element inside a `<symbol>` element, assign an identifier to this `<symbol>` and call it by creating a `<use>` element referencing to that id. Although a `<use>` element can reference to a graphical element such as a `<path>` or a `<circle>` directly, the main benefit of using a `<symbol>` element is that, unlinke `<svg>` elements, the `<symbol>` itself is never rendered, only its `<use>` reference is. This makes it extremely convinient for defining a bunch of shapes one is planing to reuse without having to worry about them being displayed unnecessary.

With all this information digested, we can now create reusable shapes by nesting them inside `<symbol>` elements and have them stretch by setting their `<viewBox>` attribute to values in percentages and `<preserveAspectRatio>` to `'none'` like we did before and then "use" them with the `<use>` element, all this inside a single SVG file.

<p data-height="265" data-theme-id="0" data-slug-hash="EbwKZo" data-default-tab="html,result" data-user="andresangelini" data-embed-version="2" data-pen-title="stretcheable SVG element" class="codepen">See the Pen <a href="https://codepen.io/andresangelini/pen/EbwKZo/">stretcheable SVG element</a> by Andrés Angelini (<a href="https://codepen.io/andresangelini">@andresangelini</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="async" src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Try resizing your browser window. Do you see how the upper side of the frame streches horizontally? Now, try also changing its width either directly on the `<symbol>` element or overrinding it on the `<use>` element. Isn't it amazing? All in all, this might seem like a trivial thing but in reality, it's a big step towards our main goal of having a full fledged responsive board. We'll repeat this same process later with all the other pieces that need to stretch.

In the [next chapter][ch-3] of our journey we will be playing around with SVG elements' transformation attribute. We will begin by try to figure out a way to invert an element so that we can reuse a give given graphic. Don't miss it!



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
