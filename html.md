---
layout: chapter
title: HTML
permalink: /html/
next: /css/
---

[HTML][html] is a markup language for describing the structure of web pages. HTML is made out of [elements][html-elements] which are marked using a start tag (`<title>`) and end tag (`</title>`). Between the start and end tags you can put text and other elements. The start tags may contain [attributes][html-attributes] (`<img src="picture.jpg">`). For some elements the end tag is not needed.


## Minimal HTML page

The following code shows the minimal structure of a HTML page.

Create a folder called `MyAwesomeSite` on your computer's desktop. This will be your project folder where to put all the files you create in this tutorial.

Copy the following HTML code into your text editor and save it as `about.html` into your project folder. Then open the file in your web browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>This is the page title</title>
</head>
<body>

<p>This is a <u>p</u>aragraph of text</p>

</body>
</html>
```

It should look like this:

![Minimal HTML page](minimal-html-page.png)

<aside class="solution">
    <a class="file" href="https://github.com/luontola/web-intro-project/blob/f96f5cf249763b65d888827387b227c2c651bdd2/about.html">about.html</a>
    <a class="diff" href="https://github.com/luontola/web-intro-project/commit/f96f5cf249763b65d888827387b227c2c651bdd2">View changes</a>
</aside>


## Basic HTML elements

Edit the page you just created to say a few things about yourself.

Create a heading with `<h1>some text</h1>`, one or more paragraphs with `<p>some text</p>` and a picture using `<img src="picture.jpg" alt="alternative text">` (if you don't have a picture of yourself, you can use [this one](ruby.png)). All visible elements like these will go between `<body>` and `</body>`. Also change the page title.

You can learn more about [`<h1>`][html-h1], [`<p>`][html-p], [`<img>`][html-img] and all other HTML elements from the [HTML element reference][html-elements] at MDN Web Docs.

![Basic HTML Elements](basic-html-elements.png)

<aside class="solution">
    <a class="file" href="https://github.com/luontola/web-intro-project/blob/e5466a8e79dcb612be4646f1a291d5760c97dcde/about.html">about.html</a>
    <a class="diff" href="https://github.com/luontola/web-intro-project/commit/e5466a8e79dcb612be4646f1a291d5760c97dcde">View changes</a>
</aside>


[html]: https://developer.mozilla.org/en-US/docs/Web/HTML
[html-elements]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element
[html-attributes]: https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes
[html-h1]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements
[html-p]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/p
[html-img]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img
