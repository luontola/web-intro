---
layout: chapter
title: CSS
permalink: /css/
next: /templates/
---

[CSS][css] is a markup language for describing the visuals of web pages. It contains numerous [properties][css-properties] for changing how the HTML elements look - their color, size, position etc.


## Add a CSS framework

To make working with CSS easier, there are CSS frameworks which have reusable solutions to common problems. In this tutorial we'll use [Pure](http://purecss.io/).

Add the following code inside your HTML page's `<head>` element to install Pure.

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/pure-min.css">
<link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/grids-responsive-min.css">
```

*Note: The `<meta charset="UTF-8">` element must be in the beginning of `<head>`, but otherwise the order of elements inside `<head>` doesn't usually matter.*

When you reload the page, you should see that the text has a bit nicer font.

![Default styles using Pure CSS](prettier-defaults.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/6ba4873b0cc4b1e8c574c8e9dc99dac516bfe03c/about.html">about.html</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/6ba4873b0cc4b1e8c574c8e9dc99dac516bfe03c">View changes</a>
</aside>


## Custom styles

In a CSS file we can in one place change the visual style of any elements on the web site. For example we can say that all headings should be red. We can also use *CSS classes* to give elements descriptive names and change their style with more precision. For example there are many pictures on the web site, but only the profile picture has a red border.

To have a place where to add our own CSS, create a file `style.css` with the following content.

```css
h1 {
    font-size: 30px;
    color: #E0330C;
}

.profile-picture {
    float: right;
    border: 1px solid #E0330C;
}
```

Add to your `<head>` a stylesheet link which points to that CSS file.

```html
<link rel="stylesheet" href="style.css">
```

This CSS defines the visual style for the `<h1>` element and `profile-picture` class. The `.` in front of `.profile-picture` means that any element with the `class="profile-picture"` attribute will have this style.

Add the `class="profile-picture"` attribute to your `<img>` tag. The `float: right` CSS property makes the picture go to the right side of any elements that follow it. To make the picture go to the right side of your page's heading, you will need to reorder your code so that the picture is before the heading.

![Custom styles](custom-styles.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/7c64178e9ec12310542aa1f7925e1df414451d04/about.html">about.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/7c64178e9ec12310542aa1f7925e1df414451d04/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/7c64178e9ec12310542aa1f7925e1df414451d04">View changes</a>
</aside>


## Layout

A good starting point for making a web site is to draw on paper a sketch of what you want the site to look like. Here is a sketch of a typical layout where the navigation menu is on the left and page content in the middle.

![Layout sketch](layout-sketch.jpg)

Then we can start converting that into code. We'll start with the structure, then position the navigation and content, and finally make it all pretty.


### Page structure

Add the following elements to your page's `<body>` and put the page content you wrote earlier inside the content section.

```html
<nav class="navigation">
    <ul>
        <li><a href="about.html">About</a></li>
        <li><a href="pictures.html">Pictures</a></li>
    </ul>
</nav>

<section class="content">
    page content goes here
</section>
```

This navigation menu is made out of an *unordered list* (`<ul>`) which contains *list items* (`<li>`) which contains links (`<a>` as in <u>a</u>nchor) to the site's pages (we'll create the pictures page later).

The `<nav>` and `<section>` elements are just "boxes" for more content. They are similar to the `<div>` element (if you happen to know some HTML), but have some semantic meaning, so for example the screen readers for blind people can be smarter about where the navigation is.

![Page structure](page-structure.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/20efe0cbb514292c31a3d43273be8fc042698540/about.html">about.html</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/20efe0cbb514292c31a3d43273be8fc042698540">View changes</a>
</aside>


### Prepare for layout fiddling

To get the page layout right, it's easier to focus on one thing at a time. We'll first focus on just positioning the navigation and page content. Afterwards we can make them pretty.

To get ready for fiddling with the positioning of layout elements, make the edges of `.navigation` and `.content` visible by adding the following CSS code to your `style.css` file.

```css
.content {
    border: 1px solid Green;
}

.navigation {
    border: 1px solid Blue;
}
```

![Layout elements highlighted](prepare-for-layout-fiddling.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/779795bc21b66edc64de0cd66d16f07c014a24a4/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/779795bc21b66edc64de0cd66d16f07c014a24a4">View changes</a>
</aside>


### Position the layout elements

Most CSS frameworks have the concept of a grid for positioning elements. The basic idea is that the page is divided into rows and columns. You can then say that how many columns wide an element is. If an element does not fit to the same row as the previous element, it will go to the next row.

Here is a layout for you. It uses Pure's [grid system][pure-grids] and [vertical menus][pure-menus]. 

```html
<div class="pure-g">
    <div class="pure-u-1 pure-u-sm-1-5">
        <nav class="navigation pure-menu">
            <ul class="pure-menu-list">
                <li class="pure-menu-item"><a class="pure-menu-link" href="about.html">About</a></li>
                <li class="pure-menu-item"><a class="pure-menu-link" href="pictures.html">Pictures</a></li>
            </ul>
        </nav>
    </div>
    <div class="pure-u-1 pure-u-sm-4-5 pure-u-md-3-5">
        <section class="content">
            page content goes here
        </section>
    </div>
</div>
```

Here is the updated CSS.

```css
.content {
    border: 1px solid Green;
    min-height: 300px;
}

.navigation {
    border: 1px solid Blue;
}

@media screen and (min-width: 35.5em) { /* same as pure-u-sm  */
    .navigation {
        margin-top: 2em;
    }
}
```

![Responsive layout in desktop size](position-the-layout-elements--desktop.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/ae9b6357120b5603b06f45b6c0d421a335181ce5/about.html">about.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/ae9b6357120b5603b06f45b6c0d421a335181ce5/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/ae9b6357120b5603b06f45b6c0d421a335181ce5">View changes</a>
</aside>

This layout is *responsive*, which means that it will adjust itself to both big and small screens - from desktops to mobile phones. Try making your browser window narrower and see how the navigation menu will move from left to top.

![Responsive layout in mobile size](position-the-layout-elements--mobile.png)

Try fiddling with the code to figure out which of the CSS classes does what. For example remove the `pure-u-sm-1-5` class and see how the page behaves now when you resize it. Then put that code back and try changing something else until you have a feel for how it works.


### Make it pretty

After the boxes are positioned the way you like, remove the placeholder borders from `.content` and `.navigation`, and focus on their visual style. You can for example change their [background][css-background], [border][css-border], [padding][css-padding] and [margin][css-margin] (padding is empty space inside the borders, margin is empty space around the borders).

*Feel free to copy the example solution as a starting point.*

If you want for example rounded corners, just google "css rounded corners" for some sample code. This kind of CSS tricks are hard to make yourself; everybody just copies them from someone else. ;-)

![Finished layout styling](make-it-pretty.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/26b92b4f8945c50ad1e5678863e0790fef48a167/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/26b92b4f8945c50ad1e5678863e0790fef48a167">View changes</a>
</aside>


[css]: https://developer.mozilla.org/en-US/docs/Web/CSS
[css-background]: https://developer.mozilla.org/en-US/docs/Web/CSS/background
[css-border]: https://developer.mozilla.org/en-US/docs/Web/CSS/border
[css-margin]: https://developer.mozilla.org/en-US/docs/Web/CSS/margin
[css-padding]: https://developer.mozilla.org/en-US/docs/Web/CSS/padding
[css-properties]: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference
[html-li]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul
[html-ul]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul
[pure-grids]: https://purecss.io/grids/
[pure-menus]: https://purecss.io/menus/
