---
layout: chapter
title: CSS
permalink: /css/
next: /templates/
---

[CSS][css] is a markup language for describing the visuals of web pages. It contains numerous [properties][css-properties] for changing how the HTML elements look like; colors, fonts, layouts etc.


## Prettier defaults

At first let's make the default text look a bit nicer by using the [Pure](http://purecss.io/) CSS library.

Add the following code inside your HTML page's `<head>` section.

```HTML
<link rel="stylesheet" href="http://yui.yahooapis.com/pure/0.6.0/pure-min.css">
```

*Note: The `<meta charset="UTF-8">` element must be in the beginning of `<head>`, but otherwise the order of elements doesn't matter.*

![Pure CSS](pure.png)

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

This CSS defines what `<h1>` elements and any elements with the attribute `class="profile-picture"` should look like. Add the `profile-picture` class to your image and it will be positioned on the right side of elements that follow it (move it above your heading and paragraphs).

![Custom styles](custom.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/fdeae81320dcc3d9fcf7a1ec8d8af6fabe196cfa)


## Prepare for layout fiddling

Most web sites contain a navigation menu for moving between pages. Let's create a navigation menu that will be on the left side of the page content and also limit the width of the page content to look nicer on wide screen.

Add the following elements to your page's `<body>` and put the page content you wrote earlier inside the content section.

```html
<nav class="navigation">
    navigation will be here
</nav>

<section class="content">
    page content goes here
</section>
```

To get ready for fiddling with CSS layouts, make the edges of the `.navigation` and `.content` elements visible by adding the following CSS code to your `style.css` file.

```css
.content {
    border: 1px solid Green;
}

.navigation {
    border: 1px solid Blue;
}
```

![Layout elements highlighted](layout-blocks-before.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/07fbf94b53f888e010c61a2c31b1fdf861684ef6)


## Get the layout right

First add the `position: absolute;` CSS property to the style definitions of `.content` and `.navigation`.

Then start experimenting with different values for the CSS properties [left][css-left], [top][css-top], [width][css-width] and [min-height][css-min-height] to get the boxes to look like shown below.

It's also good to try how the layout stretches when the page has many or few paragraphs of text, or when the browser window is wide or narrow.

![Layout elements where we want them](layout-blocks-after.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/d460ff730c7e4ccb55c1821a6fe7e8fe9fc14b9b)


## Make it pretty

After the boxes are where you want them to be, remove those placeholder borders and move on to styling the layout elements properly. You can change their [background][css-background], add some [margin][css-margin] between the edges and the page content and create nicer [borders][css-border].

For the navigation menu, create an *unordered list* (`<ul>`) which contains *list items* (`<li>`) which contains links (`<a>` as in *anchor*) to the various pages that we will be creating.

```html
<nav class="navigation">
    <ul>
        <li><a href="my-page.html">Ruby</a></li>
        <li><a href="pictures.html">Pictures</a></li>
    </ul>
</nav>
```

You'll probably want to remove those black bullet points in front of every `<li>`. Google for "css vertical navigation bar" to find out how to make it look nicer, or just copy the example solution from the link below. This kind of CSS tricks are hard to make yourself; everybody just copies them from someone else. ;-)

![Finished layout styling](pretty-layout.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/e7e4aa852afe598bf7aa609a4a6a22775992d773)


[css]: https://developer.mozilla.org/en-US/docs/Web/CSS
[css-properties]: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference
[css-position]: https://developer.mozilla.org/en-US/docs/Web/CSS/position
[css-left]: https://developer.mozilla.org/en-US/docs/Web/CSS/left
[css-top]: https://developer.mozilla.org/en-US/docs/Web/CSS/top
[css-width]: https://developer.mozilla.org/en-US/docs/Web/CSS/width
[css-min-height]: https://developer.mozilla.org/en-US/docs/Web/CSS/min-height
[css-background]: https://developer.mozilla.org/en-US/docs/Web/CSS/background
[css-margin]: https://developer.mozilla.org/en-US/docs/Web/CSS/margin
[css-border]: https://developer.mozilla.org/en-US/docs/Web/CSS/border
[html-ul]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul
[html-li]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul
[css-navigation-bar]: http://www.w3schools.com/css/css_navbar.asp
