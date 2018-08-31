### Namespacing

Now that we learned about the existence of `defs` and how to take advantage of them properly, we should start thinking about using some kind of naming system for our elements IDs, so that we can clearly recognize what is what and thus avoid any potencial name collision. So far, we have three types of elements: basic shapes (the actual base graphics, such as `<path>`), their modifications (i.e.: our reused path made to stretch) and their implementations (when we `<use>` something outside `<defs>` ). This is where [namespaces] really come in handy. Basically, we are going to prepend an abbreviated version of its category to each element name. I'll go ahead and add some usefull categories and their corresponding namespaces into a table for future reference.

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

Pretty neat, right? But we will see the true usefullness of namespaces once we have all the pieces of the board altogether.
