---
layout: page
title: HTML
permalink: /html/
---

[HTML][html] is a markup language for describing the structure of web pages. HTML is made out of [elements][html-elements] which are marked using a start tag (`<title>`) and end tag (`</title>`). The start tags may contain [attributes][html-attributes] (`<meta charset="UTF-8>`). Between the start and end tags you can put text and other elements. For some elements the end tag is optional.


## Minimal HTML page

The following code shows the minimal structure of a HTML page. Save it in a file called `my-page.html` and open it in your web browser.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>title goes here</title>
</head>
<body>

body goes here

</body>
</html>
```

It should look like this:

![Minimal HTML page](/screenshots/html-minimal.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/389109b2739774161919054e5de7818029d7cac6)


## Basic HTML elements

Edit the page you just created to say a few things about yourself. Create a heading with [`<h1>`][html-h1], one or more paragraphs with [`<p>`][html-p] and a picture using [`<img>`][html-img]. Also change the title.

![Basic HTML Elements](/screenshots/html-basics.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/e34cc4d65dafa0cbc5738cf48b21f254f820a482)

<a class="next-chapter" href="/css/">Proceed to the next chapter</a>


[html]: https://developer.mozilla.org/en-US/docs/Web/HTML
[html-elements]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element
[html-attributes]: https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes
[html-h1]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements
[html-p]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/p
[html-img]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img
