---
layout: chapter
title: CSS
permalink: /css/
next: /templates/
---

[CSS][css] is a markup language for describing the visuals of web pages. It contains numerous [properties][css-properties] for changing how the HTML elements look like (color, font, position and so on).


## Add a CSS framework

To make working with CSS faster and easier, there are CSS frameworks which have reusable solutions to common problems. In this tutorial we'll use [Pure](http://purecss.io/).

Add the following code inside your HTML page's `<head>` element.

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/pure-min.css">
<link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/grids-responsive-min.css">
```

*Note: The `<meta charset="UTF-8">` element must be in the beginning of `<head>`, but otherwise the order of elements inside `<head>` doesn't usually matter.*

When you reload the page, you will see that the text has a bit nicer default font.

![Default styles using Pure CSS](prettier-defaults.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/a6be40bb4f6f5a511ad4bb060efebd81ab961939/about.html">about.html</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/a6be40bb4f6f5a511ad4bb060efebd81ab961939">View changes</a>
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

Add into your `<head>` a stylesheet link which points to that CSS file.

```html
<link rel="stylesheet" href="style.css">
```

This CSS defines the visual style for the `<h1>` element and `profile-picture` class. The `.` in front of `.profile-picture` means that any element with the `class="profile-picture"` attribute will have this style.

Add the `class="profile-picture"` attribute to your `<img>` tag. The `float: right;` CSS property makes the picture go to the right side of elements that follow it. To make the picture be on the right side of your page's heading, you will need to move the picture's element before the heading's element.

![Custom styles](custom-styles.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/fdeae81320dcc3d9fcf7a1ec8d8af6fabe196cfa/about.html">about.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/fdeae81320dcc3d9fcf7a1ec8d8af6fabe196cfa/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/fdeae81320dcc3d9fcf7a1ec8d8af6fabe196cfa">View changes</a>
</aside>


## Layout

A good starting point for making a web site is to draw on paper a sketch of what you want the site to look like. Here is a sketch of a typical layout where the navigation menu is on the left and page content in the middle.

![Layout sketch](layout-sketch.jpg)

Then we can start converting that into code. We'll start with the structure, then the positioning of the navigation and content, and finally make it all pretty.


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
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/7c671763aafe022c0977e54143e2759cf8a9cd24/about.html">about.html</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/7c671763aafe022c0977e54143e2759cf8a9cd24">View changes</a>
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
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/5271129168f39165b420629ac1c2a41c579507c0/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/5271129168f39165b420629ac1c2a41c579507c0">View changes</a>
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

<!-- TODO: picture of page in desktop size -->

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/e500f8e79849b13001c7c694f3030951d3faa52e/about.html">about.html</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/e500f8e79849b13001c7c694f3030951d3faa52e/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/e500f8e79849b13001c7c694f3030951d3faa52e">View changes</a>
</aside>

This layout is *responsive*, which means that it will adjust itself to both big and small screens - from desktops to mobile phones. Try making your browser window narrower and see how the navigation menu will move from the left to the top.

<!-- TODO: picture of page in mobile size -->

Try fiddling with the code to figure out which of the CSS classes does what. For example remove the `pure-u-sm-1-5` class and see how the page behaves now when you resize it. Then put that code back and try removing something else until you have a feel for how it works.


### Make it pretty

After the boxes are where you want them to be, remove those placeholder borders and move on to styling the visual look of the layout elements. You can for example change their [background][css-background], [border][css-border], [padding][css-padding] and [margin][css-margin] (padding is empty space inside the borders, margin is empty space around the borders).

*Feel free to copy the example solution as a starting point.*

If you want for example rounded corners, just google "css rounded corners" for some sample code. This kind of CSS tricks are hard to make yourself; everybody just copies them from someone else. ;-)

![Finished layout styling](make-it-pretty.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/4684a6bd538e3a1f751d95f99e319a1b9d45f40c/style.css">style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/4684a6bd538e3a1f751d95f99e319a1b9d45f40c">View changes</a>
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
