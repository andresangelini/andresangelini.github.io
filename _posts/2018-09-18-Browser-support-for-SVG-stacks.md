### Browser suppor for SVG stacks

Creating a responsive UI with a single SVG using SVG stacks it's one thing, but
making it actually work across a plethora of devices and broswers it's a whole
different beast on its own. Of course, when we say "work" we mean that the site should be at least functinoal, regardless of its appearence. Even if a certain graphic cannot be displayed properly due to an incompatibility, for instance, the user should still be able to use the site without any hindrance. This is something I have intentionally avoided talking about because I thought it would be better to present the problem first, but since it plays a key role in developing a website it should always be part of the designing stage.

#### Progressive Enhancement and Graceful Degradation

There are basically two popular techiniques, or perhaps schools of thought that try to solve the problem of how to make a site functional no matter what the user is using. Both of them recognize the fact that it's not always possible to make a site funciton the same way or even work properly in every scenario and so some consessions must be made. However, their key differece is the starting point for planning out a strategy.

**Progressive Enhancement** holds the idea that one should design **starting from the lowest common denominator** and add **enhancements** as the user device and browser allow. This is the prefered technique by most developers because by focusing more in functionality rather than the shiny it helps writing cleaner code which will be easier to mantain and doesn't make the user feel left out for not using a modern device or browser.

**Graceful Degradation**, on the other hand, is the technique where you start off designing for the latest and greatest and then you accomadate your design for older, less capable devices or browsers. Many developers actually dislike this way of thinking because they wrongly believe it means adding messages encouraging the user to update or use such and such browser when something doesn't work. In all honestly, they can't be blamed for it because this practice has been widely used and abused for many years. However, that is not what **Graceful Degradation** is about. The key to implement this technique succesfully is to do whatever is necesary to **achieve the same level of funcitonality and appearence wihtout the user even noticing** when a feature is not available for his or her device or browser. When that isn't possible, then you can reduce or eliminate certain visual elements as long as any of the site functionality isn't affected in any way. Adding messages prompting the users to update their browser or device is reserved for those edge cases where they are using something so old or strange that goes beyond our target audience's requierements.

#### What did I choose and why?

When designing a website it should go without saying that you should do so with a target audience in mind and take into account their requirements and needs. That's why the **Progressive Enhancement** approach usually fits the bill better and what I usually go for. In this particular case, however, we intend to make the site look and feel just like a video game because it's geared towards companies and professionals in the video game industry who would probably be visiting the site from a high end rig either at the office or at their home, or to a lesser extent, from a modern mobile device. We are not expecting anybody to be using some old version of Explorer on Windows XP and if there was ever such a person, it wouldn't probably belong to our target audience anyway. These few reasons alone make choosing **Graceful Degradation** a no-brainer, so let's plan out how this will work out.

##### Graceful Degradation doesn't mean Excluding

Following this second approach to web design, we will start off with our Medieval Board made with a single SVG stack using all the latest features present in moder browsers and then work out how keep it visually and functionally consisten across browsers and devices. Our main goal here is to try our best not to let the user feel excluded or treated as a second class citizen.

#### The best tools for the job

There two crucial tools that we will be using to achive true ubiquitous exprerience. The first one is the sitr [Can I Use] which is basically an up to date table of support for HTML, CSS and JavaScript features across a wide range of browser vendor, versions and devices. The second is called [Cross Browser Testing] and although you need to have an account, you can access to their **Live Testing**, **Sreenshot** testing and other amazing tools with a **free trial**, which is more than enough for our needs.

After carefully testing the boards we get the following results:

- IE < 11 don't scale SVGs properly when used as background images.
- Safari and Safari iOS don't support SVG Fragment Identifiers at all.
- Chrome versions 36 to 49 only support SVG Fragment Identifiers when used inside `<img>` elements.
- Safari 6, iOS Safari 6.1 and Chrome versions 25 to 19 suupport `calc()` with `-webkit-` prefix but older versions do not support it at all.
- Safari won't display background image if background size and position aren't explicity set.
- In Safari 11 SVG pattern elements with alpha are displayed with a black background instead of transparency.
- Firefox 15 and older don't display SVG elements that use viewbox attribute.
- IE < 10 and very old version of FF, Chrome, Safari, Opera, etc., do not display SVGs properly, probably due to incompatibility issues with <pattern> elements.
- IE < 10 dont't support innerHTML on HTML elements.
- IE8 only supports the single-colon CSS 2.1 syntax (i.e. :pseudo-class).

We finally have a list of specific issues we can address to, but we now need some way to **detect** when a feature is not supported.

#### Broser Detection vs Feature Detection

Was once upon a time when the lack of standard practices in web development made Internet look more like the Wild West than anything else, developers used to use a technique called **browser detection**. As the name implies, it consisted in trying to **detect** what browser the user was using, usually with the help of JavaScript. Not only was this an invasion of user's privacy (and why it was more widely known as [browser sniffing]), but it was also proven to be highly inneffective.


[Can I Use]: https://caniuse.com/
[Cross Browser Testing]: https://crossbrowsertesting.com/
[browser sniffing]: https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/JavaScript#Using_bad_browser_sniffing_code
