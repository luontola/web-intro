---
layout: chapter
title: CSS
permalink: /css/
next: /templates/
---

[CSS][css] is a markup language for describing the visuals of web pages. It contains numerous [properties][css-properties] for changing how the HTML elements look like (color, font, position and so on).


## Prettier defaults

At first let's make the default text look a bit nicer by using the [Pure](http://purecss.io/) CSS library.

Add the following code inside your HTML page's `<head>` element.

```HTML
<link rel="stylesheet" href="http://yui.yahooapis.com/pure/0.6.0/pure-min.css">
```

*Note: The `<meta charset="UTF-8">` element must be in the beginning of `<head>`, but otherwise the order of elements inside `<head>` doesn't usually matter.*

![Default styles using Pure CSS](prettier-defaults.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/a6be40bb4f6f5a511ad4bb060efebd81ab961939)


## Custom styles

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

This CSS defines the visual style for the `h1` element and `profile-picture` *class*. The `.` in front of `.profile-picture` means that any element with the attribute `class="profile-picture"` will have this style.

Add the `profile-picture` class to your `<img>` tag, so that the `float: right;` CSS property will position it to the right of elements that follow it. To make the picture be on the right side of your page's heading, you will need to move the picture's element before the heading's element.

![Custom styles](custom-styles.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/fdeae81320dcc3d9fcf7a1ec8d8af6fabe196cfa)


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

This navigation menu is made out of an *unordered list* (`<ul>`) which contains *list items* (`<li>`) which contains links (`<a>` as in *anchor*) to the site's pages (we'll create the pictures page later).

The `<nav>` and `<section>` elements are just "boxes" for more content. They are similar to the `<div>` element (if you happen to know some HTML), but have some semantic meaning, so for example the screen readers for blind people can be smarter about where the navigation is etc.

![Page structure](page-structure.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/7c671763aafe022c0977e54143e2759cf8a9cd24)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/5271129168f39165b420629ac1c2a41c579507c0)


### Position the layout elements

First add the `position: absolute;` CSS property to the style definitions of `.content` and `.navigation`. [Absolute position][css-position] means that the position of the element is relative to the browser window, instead of other HTML elements (which is the default). Don't worry about understanding this yet.

Then start experimenting with different values for the CSS properties [left][css-left], [top][css-top] and [width][css-width] to get the boxes to look like shown below. Instead of [width][css-width] you can also use [right][css-right] to make it stretch based on the browser window size.

It's a good practice to try out how the layout stretches when the page has many or few paragraphs of text, or when the browser window is wide or narrow. You can use [min-height][css-min-height] to enforce a minimum height for an element when it has only a little content.

![Layout elements positioned](position-the-layout-elements.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/a4680b89489fe8d150d6c3bb521ae7f77871bd68)


### Make it pretty

After the boxes are where you want them to be, remove those placeholder borders and move on to styling the visual look of the layout elements. You can for example change their [background][css-background], [border][css-border], [padding][css-padding] and [margin][css-margin] (padding is empty space inside the borders, margin is empty space around the borders).

For the navigation menu, just copy the following CSS as a starting point. If you like, you can google "css vertical navigation bar" for more examples. This kind of CSS tricks are hard to make yourself; everybody just copies them from someone else. ;-)

```css
.navigation ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.navigation li {
    font-size: 20px;
    line-height: 30px;
}

.navigation a {
    text-decoration: none;
    color: #E0330C;
}

.navigation a:hover {
    color: #F09986;
}
```

![Finished layout styling](make-it-pretty.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/4684a6bd538e3a1f751d95f99e319a1b9d45f40c)


[css]: https://developer.mozilla.org/en-US/docs/Web/CSS
[css-properties]: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference
[css-position]: https://developer.mozilla.org/en-US/docs/Web/CSS/position
[css-left]: https://developer.mozilla.org/en-US/docs/Web/CSS/left
[css-top]: https://developer.mozilla.org/en-US/docs/Web/CSS/top
[css-width]: https://developer.mozilla.org/en-US/docs/Web/CSS/width
[css-right]: https://developer.mozilla.org/en-US/docs/Web/CSS/right
[css-min-height]: https://developer.mozilla.org/en-US/docs/Web/CSS/min-height
[css-background]: https://developer.mozilla.org/en-US/docs/Web/CSS/background
[css-margin]: https://developer.mozilla.org/en-US/docs/Web/CSS/margin
[css-padding]: https://developer.mozilla.org/en-US/docs/Web/CSS/padding
[css-border]: https://developer.mozilla.org/en-US/docs/Web/CSS/border
[html-ul]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul
[html-li]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul
[css-navigation-bar]: http://www.w3schools.com/css/css_navbar.asp
