### Assembling the board from an SVG with SASS

Much like when we found ourselves with a really complex SVG structure and decided to organize it using `<defs>`, namespacing and some styling, we are facing one again a bit of a problem right from the start. If we take a look back at our SVG file we can see that it's composed from several pieces which in some cases, have two variations; one for the **bulletin-board** and another for the **sign-board**. This means there will be a myriad of variables, modifications, and edge cases that our CSS will have to handle, making into a such a mess that it will be almost impossible to maintain.

This makes the perfect scenario for introducing a new tool to our project; a CSS pre-processor. This time will go with [SASS] but any other would do just fine.





[SASS]: https://sass-lang.com/
