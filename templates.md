---
layout: chapter
title: Templates
permalink: /templates/
next: /databases/
---

In this chapter you will learn how to create a web server with Ruby, and how to use templates and a bit of programming to avoid repeating yourself. You will also get a taste of creating dynamic content, i.e. the page content is not always the same when somebody visits it.


## Pictures page

Last time we created a navigation menu with a link to the `pictures.html` page, but that page doesn't yet exist. Let's create it now.

Make a copy of `about.html`, name it `pictures.html`, and change its title and heading. Find about a dozen images which you like from [Public Domain Pictures](http://www.publicdomainpictures.net/) and add their thumbnails as images on your `pictures.html` using the [`<img>`][html-img] tag.

```html
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg">
<img class="album-photo" src="http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg">
...
```

As shown above, give the images a unique class and use CSS to make them look like a photo album. In case the pictures are of different size, you can set their [height][css-height]; the web browser will set their width automatically to maintain the picture's aspect ratio.

The following example uses the CSS properties [border][css-border], [background-color][css-background-color], [margin][css-margin], [padding][css-padding] and [box-shadow][css-box-shadow].

![Pictures page with some pictures](pictures-page.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/b8ab72533a34efa609a47653ba899f57e802f135)


## Don't Repeat Yourself

In programming we have a principle called [Don't Repeat Yourself (DRY)][dry] which says that *"every piece of knowledge must have a single, unambiguous, authoritative representation within a system."* If we follow that principle, our code will be easier to understand and modify.

Our little web site is already breaking the DRY principle. Both of our pages are repeating the HTML for the layout and navigation. If we would add another page, we would have to change the navigation menu in many different HTML files. Additionally the pictures page has lots of similar `<img>` tags with just the URL changing. To solve this problem, we will introduce *templates*, so that all HTML code will be written once and only once.


## Run a web server

This far we have created our site with just static HTML pages, but in order to use templates, we must create a *web server* which will dynamically generate HTML pages. Your web browser will communicate with the web server and show the HTML pages which the web server generates.

We will use the [Ruby][ruby] programming language and the [Sinatra][sinatra] web framework to create the web server. Follow the [installation guide](/install/) to install Ruby and Sinatra if you haven't yet done so.

Create a file called `app.rb` and put the following code in it. This minimal web application contains a single *route* for the *path* `/hi` which will just generate a page with the text *Hello World!*

```ruby
require 'sinatra'
require 'sinatra/reloader'

get '/hi' do
  "Hello World!"
end
```

Then run the command `ruby app.rb` in your terminal. It should start a web server on your own machine in port 4567. To view it, visit <http://localhost:4567/hi> in your web browser. It should show the *Hello World!* text.

![Your first web server is working](run-a-web-server.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/85703bad1a3eef6e6a9ac84781067f6da90a1712)

Finally stop the web server by pressing `Ctrl+C` in your terminal. Try refreshing the page in your web browser. It should now show an error message because there is no more a web server to connect to.

To start the server again without having to type the command, press `Up` and then `Enter`.


## Static site on a web server

As a first step towards serving our web site through Sinatra, we can use Sinatra's ability to [serve static files][sinatra-static] to serve our existing HTML and CSS files unmodified.

Inside the same folder as `app.rb`, create a new folder called `public` and move your HTML and CSS files and pictures there. Then start your web server with `ruby app.rb`, visit <http://localhost:4567/pictures.html> in your web browser and make sure that your site looks the same as before, but this time it's being served to you by a web server the same way as all web sites on the Internet.

*Note: If the full path of your project folder contains non-English letters, such as åäö, Sinatra will fail to serve your static files. In that case move your project folder for example to `C:\project`*

![Your web site on a web server](static-site-on-a-web-server.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/305b7ef7f8eb223e24cc9944947bb890a2898b91)


## Fix the front page

You should make sure that people can visit your web site through its base URL, the same way as all web sites on the Internet, instead of having to know one of its sub pages.

First try going to <http://localhost:4567/> on your current site. It should show an error message "Sinatra doesn’t know this ditty." Let's fix that.

Change `app.rb` to [redirect][sinatra-redirect] the path `/` to `/about.html` by adding the following route.

```ruby
get '/' do
  redirect to('/about.html')
end
```

Check that when you visit <http://localhost:4567/>, your web browser immediately goes to the <http://localhost:4567/about.html> address.

[View solution](https://github.com/orfjackal/web-intro-project/commit/6390abceb5e2239c29da964944729686dab06888)


## Templates

The idea of *templates* is to sprinkle your HTML code with some placeholder marks where you can easily insert some dynamic content. We will use the [ERB][erb] templating *library* which is included in Ruby's *standard library* and which Sinatra also supports.

Create a `views` folder inside your project folder (next to the `public` folder). This is the folder where Sinatra expects to find all your template files.

Copy `public/about.html` to the `views` folder and name it `layout.erb`. In `views/layout.erb`, replace all page content (i.e. the stuff between `<section class="content">` and `</section>`) with just `<%= yield %>`

Copy `public/about.html` to `views/about.erb` and remove everything except the page content (i.e. the opposite of what you did in `views/layout.erb`). Copy `public/pictures.html` to `views/pictures.erb` and do the same thing. Remove `public/about.html` and `public/pictures.html`.

Add the following routes to `app.rb` to [render][sinatra-templates] the above mentioned ERB templates.

```ruby
get '/about.html' do
  erb :about
end

get '/pictures.html' do
  erb :pictures
end
```

Check that all the pages still look the same as before in your web browser. The difference is that now all the common HTML for the layout and navigation is in a single place, so changing it and adding new pages will be easier. Templates also make it possible to have dynamic content, as we will see next.

[View solution](https://github.com/orfjackal/web-intro-project/commit/42995902c6a7cda88666eda93a9af73b0655a619)


## Lottery

You might have seen on the Internet many sites which say "Click here to find out your lucky number". Here is how you can make your own using a couple of lines of code and templates.

In `app.rb`, change the `/about.html` route to contain the following code.

```ruby
get '/about.html' do
  backstreet_boys = ["A.J.", "Howie", "Nick", "Kevin", "Brian"]
  @who_i_marry = backstreet_boys.sample
  erb :about
end
```

The above code first creates a list of the names of all Backstreet Boys and stores it in the `backstreet_boys` *variable*. The next lines calls the `sample` *method* on the list, which will randomly pick one of the items in the list, and then the code stores it in the `@who_i_marry` variable. Because the variable name starts with `@`, also the template can access that variable.

In `views/about.erb`, add the following code to show the value of `@who_i_marry` on the page.

```erb
<p>I like Backstreet Boys a lot. I'm going to marry <%= @who_i_marry %>!</p>
```

Try reloading the about page in your web browser multiple times. The name should change randomly every time that the page is reloaded.

![Lottery result is shown](lottery.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/c994a887276f8f15c413d73906d4f55c8d1fa0e2)


## Front page at the root

Normally the front page of every web site is at the root path (`/`), but on our web site it's instead at the `/about.html` path. We can solve that by rendering the about template at the root path instead of doing a redirect.

In `app.rb`, remove the old `/` route and change the `/about.html` route to be the new `/` route.

In `views/layout.erb`, change the link to the about page from `<a href="about.html">` to `<a href="/">`.

Check that when you visit <http://localhost:4567/>, your web browser stays at the <http://localhost:4567/> address and shows the front page.

![Front page at the root](front-page-at-the-root.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/ef3cfe65226c4437fed66addcad1d765ddd103af)


## Album photo template

In addition to inserting single items to a template, as we did a moment ago, we can insert a list of items and repeat a piece of the template for each of them. To demonstrate that, we will remove the repetition of multiple image tags on the pictures page. Using templates might seem overkill for simple image tags, but imagine that each image will also have a description, like button and a link to the full-sized picture, in which case avoiding the repetition becomes very important.

Change your `/pictures.html` route to store a list of the picture URLs in a variable.

```ruby
get '/pictures.html' do
  @picture_urls = [
    "http://www.publicdomainpictures.net/pictures/50000/t2/cat-looking-up.jpg",
    "http://www.publicdomainpictures.net/pictures/30000/t2/cat-in-the-city-5.jpg",
    "http://www.publicdomainpictures.net/pictures/30000/t2/annoyed-cat.jpg",
  ]
  erb :pictures
end
```

In `views/pictures.erb`, use a *for loop* to repeat the image tag for each url in the list.

```erb
<% for url in @picture_urls %>
<img class="album-photo" src="<%= url %>">
<% end %>
```

Now check that the pictures page still looks the same as before. Also try adding a couple more pictures to make sure that the template works how it should.

[View solution](https://github.com/orfjackal/web-intro-project/commit/f22f9b7963c8793587c787a970163fbbbf24e8af)


## Show all pictures from a folder

There is still some work involved in adding the picture URLs by hand to the list in the route. We can simplify that by making our application generate the list automatically based on what pictures there are in a folder.

Create a `pictures` folder under the `public` folder and save there all the pictures in your list.

Change your `/pictures.html` route to generate a list of the picture URLs based on the contents of the `public/pictures` directory.

```ruby
get '/pictures.html' do
  @picture_urls = Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
  erb :pictures
end
```

Check that the pictures page still works. Try adding a couple more pictures - much easier now, isn't it?

[View solution](https://github.com/orfjackal/web-intro-project/commit/0917ff1f02f511f0d3a456761013c9562de8912f)


### How does this code work?

The above code combines multiple operations to achieve the desired result. Always when copying code from the Internet, you should try to understand what it does. One way is to read the documentation, in this case for the [glob][ruby-glob], [map][ruby-map] and [sub][ruby-sub] methods. Another way is to run pieces of the code interactively using a [REPL][repl] and check their result. Instead of a REPL you can also [print things to the terminal][ruby-printing] when you run your application.

*The following is advanced information. Feel free to skip it.*

You can start a Ruby REPL by running the `irb` command in your terminal. When you type there a line of Ruby code and press Enter, it will print the result of that code. Here is an example session of experimenting what the above code does.

```
$ irb
irb(main):001:0> Dir.glob('public/pictures/**')
=> ["public/pictures/annoyed-cat.jpg", "public/pictures/cat-1373445873hvw.jpg", "public/pictures/cat-1382017414PaD.jpg", "public/pictures/cat-hunting.jpg", "public/pictures/lieblingskater-44421287869401VYcy.jpg"]
irb(main):002:0> "public/pictures/annoyed-cat.jpg".sub('public', '')
=> "/pictures/annoyed-cat.jpg"
irb(main):003:0> Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
=> ["/pictures/annoyed-cat.jpg", "/pictures/cat-1373445873hvw.jpg", "/pictures/cat-1382017414PaD.jpg", "/pictures/cat-hunting.jpg", "/pictures/lieblingskater-44421287869401VYcy.jpg"]
irb(main):004:0> exit
```


## Unique titles for every page

Earlier when we started using templates, our web site didn't anymore have unique titles for every page. Now that we know how use variables in templates, we can give each page its own title.

In `views/layout.erb`, get the page title from the `@title` variable.

```html
<title><%= @title %></title>
```

Add to each of your routes the code for setting the title.

```ruby
@title = "type the title here"
```

Visit every page on your site to make sure that they still work and that they have different page titles.

![Unique page titles](unique-titles-for-every-page.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/ac7b434fd1ecfd942b284e7b347fa611a106a196)


## Pages for individual pictures

It would be nice to be able to click a picture on the pictures page to see that picture in full size and also allow people to write comments for that picture. For that we will need to have separate pages for every picture. Because our application can have an unlimited number of pictures, it's not possible to create separate routes for every picture, but we need a route to handle *all* the pictures.

In `views/pictures.erb`, make each of the pictures a link to another page. The target of the link will be the URL of the picture but with the file extension changed to `.html` (this code does it using [regular expressions][ruby-regexp]).

```erb
<% for url in @picture_urls %>
<a href="<%= url.sub(/\.\w+$/, '.html') %>"><img class="album-photo" src="<%= url %>"></a>
<% end %>
```

In `app.rb`, add the following route which uses [named parameters][sinatra-routes]. The `:picture` will match anything in that position of the path, and will make it accessible as `params['picture']`.

```ruby
get '/pictures/:picture.html' do
  "Here will be a page for " + params['picture']
end
```

Go to your pictures page and click some of the pictures there. Notice how the text shown on the page is related to page's address.

![Placeholder page for a picture](pages-for-individual-pictures.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/1ecac778838c16a0e8dc9ab452dc5c05c2ec1f81)


## Picture page template

Create `views/picture.erb` with the following content.

```erb
<img class="full-photo" src="<%= @picture_url %>">

<p><a href="pictures.html">Return to album</a></p>
```

Make the route render that template.

```ruby
get '/pictures/:picture.html' do
  @title = "Picture"
  @picture_url = params['picture'] + '.jpg'
  erb :picture
end
```

Visit a few picture pages pictures and check that they show the picture. You'll notice that the CSS and some links don't work, so that needs to be solved next.

![Broken page for a picture](picture-page-template.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/1d4eae230df76cc43c088d175ef8a1ffbe19cf37)


## Fix relative links

On a picture page, if you click the "Pictures" or "Return to album" link, it will take your browser to <http://localhost:4567/pictures/pictures.html> instead of <http://localhost:4567/pictures.html>. Similarly it tries to load your CSS from <http://localhost:4567/pictures/style.css> instead of <http://localhost:4567/style.css> (you can see this with your web browser's [developer tools][browser-developer-tools]). However, the link to the front page still works.

The reason has to do with how relative links work. Read [Absolute vs. Relative Paths/Links][absolute-vs-relative-paths] for an explanation. Absolute links start with `http://` or similar. Other links are relative to the page where the link appears. To make a link point to the same target regardless of the page where it appears in your site, you should prefix the link with `/`, which makes it relative to the site's root path instead of the current page's path.

Change all relative links in `views/layout.erb` and `views/picture.erb` to start with the `/` character. Check that the picture pages look right and all the links work.

![Working page for a picture](fix-relative-links.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/f3643b362c18b4d8d74f729965feda87210054da)


## Don't guess the picture's file extension

Not all pictures have the `.jpg` file extension. There are `.png`, `.gif` and other image types as well. But currently the `/pictures/:picture.html` route has been hard-coded to work with only `.jpg` images.

A solution for that is to check the contents of the `public/pictures` folder to find out the actual file extension. This can be implemented by taking advantage of the code that already lists all pictures in the folder, so we will extract that code into a new method so that we can reuse it.

In `app.rb`, add the methods `picture_urls` and `find_picture_url` as shown below, and change your routes to use them to set `@picture_urls` and `@picture_url`.

```ruby
get '/pictures.html' do
  @title = "Lovely Pictures"
  @picture_urls = picture_urls
  erb :pictures
end

get '/pictures/:picture.html' do
  @title = "Picture"
  @picture_url = find_picture_url(params['picture'])
  erb :picture
end

def picture_urls
  Dir.glob('public/pictures/**').map { |path| path.sub('public', '') }
end

def find_picture_url(basename)
  picture_urls.select { |path| File.basename(path, '.*') == basename }.first
end
```

[View solution](https://github.com/orfjackal/web-intro-project/commit/ec8659f804ecc113d0682f5a4b5476762ebf8f3b)


## Error page when the picture is not found

Currently if you go to a picture page that doesn't exist, for example <http://localhost:4567/pictures/foo.html>, it shows a page with the picture not working. But instead of that a well behaving site should give an error message that the page was not found.

In HTTP there are a bunch of numeric status codes, of which the code 404 means that a page was not found. In Sinatra we can produce that status code with `halt 404`. Additionally we can create a `not_found` route to produce an error message page (otherwise the page is empty). It could produce a full web page, but a simple "Page Not Found" text is enough.

Update your `/pictures/:picture.html` route to be as follows.

```ruby
get '/pictures/:picture.html' do
  @title = "Picture"
  @picture_url = find_picture_url(params['picture']) or halt 404
  erb :picture
end

not_found do
  "Page Not Found"
end
```

Check that <http://localhost:4567/pictures/foo.html> shows an error message. Also use the [developer tools][browser-developer-tools] to check that the status code for that page is 404.

![Page not found error shown with developer tools](error-page-when-the-picture-is-not-found.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/0129a3f6e25fa72d5ebf9e31c561795a81701385)


[html-img]: https://developer.mozilla.org/en/docs/Web/HTML/Element/img
[css-width]: https://developer.mozilla.org/en-US/docs/Web/CSS/width
[css-height]: https://developer.mozilla.org/en-US/docs/Web/CSS/height
[css-background-color]: https://developer.mozilla.org/en-US/docs/Web/CSS/background-color
[css-margin]: https://developer.mozilla.org/en-US/docs/Web/CSS/margin
[css-padding]: https://developer.mozilla.org/en-US/docs/Web/CSS/padding
[css-border]: https://developer.mozilla.org/en-US/docs/Web/CSS/border
[css-box-shadow]: https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow
[dry]: http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself
[ruby]: https://www.ruby-lang.org/
[sinatra]: http://www.sinatrarb.com/
[sinatra-static]: http://www.sinatrarb.com/intro.html#Static%20Files
[sinatra-redirect]: http://www.sinatrarb.com/intro.html#Browser%20Redirect
[sinatra-routes]: http://www.sinatrarb.com/intro.html#Routes
[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[absolute-vs-relative-paths]: http://www.coffeecup.com/help/articles/absolute-vs-relative-pathslinks/
[erb]: http://www.stuartellis.eu/articles/erb/
[ruby-read]: http://docs.ruby-lang.org/en/2.2.0/IO.html#method-c-read
[ruby-glob]: http://docs.ruby-lang.org/en/2.2.0/Dir.html#method-c-glob
[ruby-map]: http://docs.ruby-lang.org/en/2.2.0/Enumerable.html#method-i-map
[ruby-sub]: http://docs.ruby-lang.org/en/2.2.0/String.html#method-i-sub
[ruby-hash]: http://docs.ruby-lang.org/en/2.2.0/Hash.html
[ruby-printing]: http://zetcode.com/lang/rubytutorial/io/
[ruby-regexp]: http://ruby-doc.org/core-2.2.0/Regexp.html
[repl]: https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop
[browser-developer-tools]: https://developer.mozilla.org/en-US/Learn/Discover_browser_developer_tools
