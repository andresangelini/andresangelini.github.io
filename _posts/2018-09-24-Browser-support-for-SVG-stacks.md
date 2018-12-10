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

## Browser suppor for SVG stacks

Creating a responsive UI with a single SVG using SVG stacks like the one we just finished in the [previous post][ch-13] is one thing, but
making it actually work across a plethora of devices and broswers it's a whole
different beast on its own. Of course, when we say "work" we mean that the site should be at least functinoal, regardless of its appearence. Even if a certain graphic cannot be displayed properly due to an incompatibility, for instance, the user should still be able to use the site without any hindrance. This is something I have intentionally avoided talking about because I thought it would be better to present the problem first, but since it plays a key role in developing a website it should always be a part of the designing stage.

## Progressive Enhancement and Graceful Degradation

There are basically two popular techiniques, or perhaps schools of thought that try to solve the problem of how to make a site functional no matter what the user is using. Both of them recognize the fact that it's not always possible to make a site funciton the same way or even work properly in every scenario and so some consessions must be made. However, their key differece is the starting point for planning out a strategy.

**Progressive Enhancement** holds the idea that one should design **starting from the lowest common denominator** and add **enhancements** as the user device and browser allow. This is the prefered technique by most developers because by focusing more in functionality rather than the shiny it helps writing cleaner code which will be easier to mantain and doesn't make the user feel left out for not using a modern device or browser.

**Graceful Degradation**, on the other hand, is the technique where you start off designing for the latest and greatest and then you accomadate your design for older, less capable devices or browsers. Many developers actually dislike this way of thinking because they wrongly believe it means adding messages encouraging the user to update or use such and such browser when something doesn't work. In all honestly, they can't be blamed for it because this practice has been widely used and abused for many years. However, that is not what **Graceful Degradation** is about. The key to implement this technique succesfully is to do whatever is necesary to **achieve the same level of funcitonality and appearence wihtout the user even noticing** when a feature is not available for his or her device or browser. When that isn't possible, then you can reduce or eliminate certain visual elements as long as any of the site functionality isn't affected in any way. Adding messages prompting the users to update their browser or device is reserved for those edge cases where they are using something so old or strange that goes beyond our target audience's requierements.

## What did I choose and why?

When designing a website it should go without saying that you should do so with a target audience in mind and take into account their requirements and needs. That's why the **Progressive Enhancement** approach usually fits the bill better and what I usually go for. In this particular case, however, we intend to make the site look and feel just like a video game because it's geared towards companies and professionals in the video game industry who would probably be visiting the site from a high end rig either at the office or at their home, or to a lesser extent, from a modern mobile device. We are not expecting anybody to be using some old version of Explorer on Windows XP and if there was ever such a person, it wouldn't probably belong to our target audience anyway. These few reasons alone make choosing **Graceful Degradation** a no-brainer, so let's plan out how this will work out.

## Graceful Degradation doesn't mean Excluding

Following this second approach to web design, we will start off with our Medieval Board made with a single SVG stack using all the latest features present in moder browsers and then work out how keep it visually and functionally consisten across browsers and devices. Our main goal here is to try our best not to let the user feel excluded or treated as a second class citizen.

## The best tools for the job

There two crucial tools that we will be using to achive true ubiquitous exprerience. The first one is the sitr [Can I Use] which is basically an up to date table of support for HTML, CSS and JavaScript features across a wide range of browser vendor, versions and devices. The second is called [Cross Browser Testing] and although you need to have an account, you can access to their **Live Testing**, **Sreenshot** testing and other amazing tools with a **free trial**, which is more than enough for our needs.

After carefully testing the boards we get the following results:

- IE < 11 don't scale SVGs properly when used as background images.
- Safari and Safari iOS don't support SVG Fragment Identifiers at all.
- Chrome versions 36 to 49 only support SVG Fragment Identifiers when used inside `<img>` elements.
- Safari 6, iOS Safari 6.1 and Chrome versions 25 to 19 suupport `calc()` with `-webkit-` prefix but older versions do not support it at all.
- Safari won't display background image if background size and position aren't explicity set.
- In Safari 11, alpha transparencies in `background-image`s are rendered as a black if `background-repeat` or SVG's `pattern` element are used.
- Firefox 15 and older don't display SVG elements that use viewbox attribute.
- IE < 10 and very old version of FF, Chrome, Safari, Opera, etc., do not display SVGs properly, probably due to incompatibility issues with <pattern> elements.
- IE < 10 dont't support innerHTML on HTML elements.
- IE8 only supports the single-colon CSS 2.1 syntax (i.e. :pseudo-class).

We finally have a list of specific issues we can address to, but we now need some way to **detect** when a feature is not supported.

## Browser Detection vs Feature Detection

Was once upon a time when the lack of standard practices in web development made Internet look more like the Wild West than anything else, developers used to use a technique called **browser detection**, also known as [browser sniffing]. As the name implies, it consisted in using JavaScript to **detect** what browser a site was visited from. However, this method could only go so far
as to detect a browser's vendor but not its version, which meant having to prevent all versions of a browser from using a certain feature just because some of its versions didn't support it. This not only lead many websites to have poorly mantained code but also pushed browsers to implement different measures to pretend to be something else, rendering them completely untrustworthy and nailing in the coffing for this kind of practice.

A more intelligent approach and the one more widely accepted now is to implement some kind of **Feature Detection** by testing if a feature actually works also with the help of JavaScript. This has the advantage of being browser independent which means that you no longer need to update your code with each new browser release. Fortunately for us, we don't need to write our own detections.

## Let's use something more modern

The standard for implementing **Feature Detection** nowadays is using [Modernizr], which is a collection of JavaScript code or "detects", as they call them, that run on page load to let you cater the specific needs of different users.

One of the best things about [Modernizr] is that instead of having to download and install all in one package, you can [build][Modernizr download] your own package and add more modules as you need them. You will get a `modernizr-custom.min.js` file which you can reference to from your `index.html`. For brevety's sake, I renamed my custom build just `modernizr.js` and placed it inside a new folder called `scripts`.

## Not everything is perfect

Let's open `boards.scss` and try to solve our first issue:

> IE < 11 don't scale SVGs properly when used as background images.

Looking at the [download][Modernizr download] section of Modernizr you will see there is no **detect** specifically for such a problem. Of course, we could try to write our own but that's definetely time consumig and, in fact, is such a frequent issue that we would soon find ourselves spending more time writing **detects** than getting any actual work done. However, there is something else that we can do, and that is looking at [Can I Use] and see if there is another feature with more or less the same support accross browsers that has a **detect** written for it. Naturally, this technique is far from perfect because we could potentionally be preventing certain users from enjoying features they would otherwise be prefectly able to run if one of the them losses support in the future while the other don't. However, the likelihood of that happening is so miniscule that it's well worth the risk.

Let see then what other feature we can find to use as a **detect** to cover those scenarios where SVGs aren't properly scaled when used as `background-image`s. Of all the features that have a [modernizr] **detect** available, CSS [pointer-events] (for HTML) seem to be the best, so we select **CSS Pointer Events** module and **build** our `modernizer.js` file and finally reference to it from `index.html` like this:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!--  Meta  -->
   <meta charset="UTF-8" />
   <title>Medieval Board</title>

    <!--  Styles  -->
    <link rel="stylesheet" href="styles/index.processed.css">
  </head>
  <body>

  <!-- Scripts -->
    <script src="scripts/modernizr.js"></script>
  </body>
</html>
```

[Modernizr] will add a `no-csspointerevents` to the `<html>` element, also know as the [root], when it **detects** no support **CSS pointer events**. So all we need to do now is to go back to our `boards.scss` file and add a new rule for all elements with `.no-csspointerevents` and also `.board--type-bulletin` or `.board--type-sign` assigned to them.

```scss
.board{
  // Some declarations.

  &--type-bulleting {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {

    }
  }

  &--type-sign {
    // More declarations.

    /* IE < 11 don't scale SVGs properly when used as background images. Since IE < 11 don't support CSS pointer-events, use it to detect these versions. */
    @at-root .no-csspointerevents & {

    }
  }
}
```

The `@at-root` directive let us go back to the [root] of the document or the `<html>` element while staying inside another rule in `scss`. If we were to write these rules in vanilla CSS, it would look like this:

```css
.no-csspointerevents .board--type-bulletin {}
.no-csspointerevents .board--type-sign {}
```

And our HTML markup:

```html
<div class='board board--type-bulletin bulletin'></div>
<div class='board board--type-sign sign'></div>
```

Now that we have a better idea of how to compansate for the lack of **detects** for a given feature, let's make a brief stop here to give our poor overloaded brains some time to digest and process all this amazing new knowledge we have just acquired. In the [next post][ch-15], we will introduce yet another Sass feature: **mixins**.



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
[Can I Use]: https://caniuse.com/
[Cross Browser Testing]: https://crossbrowsertesting.com/
[browser sniffing]: https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/JavaScript#Using_bad_browser_sniffing_code
[Modernizr]: https://modernizr.com/
[Modernizr download]: https://modernizr.com/download
[pointer-events]: https://caniuse.com/#feat=pointer-events
[root]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html
