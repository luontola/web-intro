---
layout: chapter
title: Databases
permalink: /databases/
next: /security/
---

Databases are used by applications to store data in a safe place. Think of them as Excel sheets with possibly millions of rows of data, but with more powerful tools for modifying and calculating things based on the data.

This this chapter we will implement a commenting feature. Later it could be expanded to having comments for each of the pictures you have on your site, but to start simple let's first focus on just writing a guestbook where visitors can leave their messages.


## Web forms

Web applications can receive input from the user though [web forms][web-form]. A web form is marked by a [`<form>`][html-form] element which can contain for example [`<input>`][html-input] text fields for the user to write into and a [`<button>`][html-button] for submitting the form.

Create a guestbook page with the following form.

```html
<form action="/add-comment" method="post" class="add-comment">
    <label for="name">Name</label>
    <br><input type="text" name="name">
    <br><label for="comment">Comment</label>
    <br><textarea type="text" name="comment"></textarea>
    <br><button type="submit">Send</button>
</form>
```

![Form for writing comments](/screenshots/comments-form.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/796018db7c92e03d10b72a453632a95743d4e572)


### Receiving form parameters

Write something into the form and press the Submit button. Sinatra should give you an error page because the necessary route is missing. Go add that route to your application and try to submit the form again. It should now work.

Did you notice that the `<form>` element had a `method="post"` attribute and that the route declaration also mentioned `post`? This refers to the [HTTP request methods][http-methods] of which the most common ones are GET and POST. GET is meant for reading pages and it should not change the application's state, so you can safely do multiple GET request to a page. POST is used for sending data to the page and it may be used to change the application's state.

To see the parameters which you typed into the form, change your `post '/add-comment'` route to be as shown below. The `puts` will print the parameters and then it will redirect the visitor back to the guestbook.

```ruby
post '/add-comment' do
  puts params
  redirect '/guestbook.html'
end
```

Now when you submit a form, it should print to the terminal where your application is running something like `{"name"=>"Ruby", "comment"=>"This is fun!"}`.

[View solution](https://github.com/orfjackal/web-intro-project/commit/b4e967b6399704cf96ad541c231ebb6b7dac2aa3)


## Storing comments in a text file

Let's keep things simple and implement as much as we can without a database. That way you will also see how a database makes many things better. At first we'll just save the comments into a text file.


### Writing to the text file

Use the following code in your route to append the comment and the name of its author into a text file.

```ruby
File.open('comments.txt', 'a') do |f|
  f.puts(params['name'] + ': ' + params['comment'])
end
```

Try to submit some comments to the guestbook and check that they are added to `comments.txt`.

[View solution](https://github.com/orfjackal/web-intro-project/commit/93838d36bd66d22a3d6363abfb49dba0ef888498)


### Reading from the text file

Now let's show the comments on the guestbook page. Use the following code to read the contents of the text file (it takes into consideration the case that the text file doesn't yet exist). Then give it as a local variable to the guestbook template and show it there.

```ruby
comments = IO.read('comments.txt') if File.exist?('comments.txt')
```

![Show comments, bare bones solution](/screenshots/comments-txt.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/1db02859480c0a2bccfcf748d9380ef66d3803a9)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Rendering comments using templates

To make the comments look better, let's render them as HTML using templates. First create a `comment.erb` template for a single comment as shown below.

```html
<p>
  <span class="comment-author"><%= name %> wrote on <%= date.strftime('%Y-%m-%d %H:%M') %>:</span>
  <br><span class="comment-message"><%= comment %></span>
</p>
```

*Note: This template has a security vulnerability, but we'll address that in a [later chapter](/security/).*

Then in your `/add-comment` route write the rendered HTML to the file using the following code.

```ruby
comment = erb :comment, :layout => false, :locals => {
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
}
f.puts(comment)
```

The [layout option][sinatra-templates] had to be set to `false` because otherwise Sinatra will try to wrap the template in `layout.rb`. [DateTime][ruby-datetime] objects can be rendered as text using [strftime][ruby-strftime] - see its documentation if you want the date to be in a different format.

Try adding some comments and write some CSS to make it look the way you like it.

![Comments rendered using templates](/screenshots/comments-html.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/0098815dfb61eae146992e1dd33e45b552ad9255)


## Storing comments in memory

Though the comments are now visible, they are very limited. For example we cannot easily edit or remove comments, or change the template for existing comments.

As a step towards using a database, we will store the comments in application memory. That way the comments will disappear when the application is stopped, but we'll be able do the necessary changes to our templates. After that switching to a real database will be a small step.

Create an [array][ruby-array] for comments as a *global variable* (in Ruby signified by the `$` prefix) by adding the following code outside your routes.

```ruby
$comments = []
```

Then in your `/add-comment` route, use the following code to append new comments as hashes to the `$comments` array.

```ruby
$comments << {
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
}
```

Finally update the template `guestbook.rb` to render all the comments similar to how we rendered all the pictures in `pictures.erb`. You can access the elements of the comment hashes similar to how we accessed the elements of the `PAGES` hash.

Check that adding comments works.

[View solution](https://github.com/orfjackal/web-intro-project/commit/243a1734f0b048eea92efc702a56767d88263034)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Storing comments in a database

The commenting feature is now working nicely and the code is easy to change, but when you restart the server (try it), all the comments will be lost. To keep the data save and the code maintainable, a database is needed.

We will use the [SQLite](https://www.sqlite.org/) database, the [DataMapper](http://datamapper.org/) library for accessing SQLite in Ruby, and [DB Browser for SQLite](http://sqlitebrowser.org/) as a UI for seeing inside the database. If you haven't yet installed them, follow the [installation guide](/installation/) and come back here.


### Creating the database

Add the following code to the beginning of your application. It will create a file `my-database.db` where all your data will be saved. In that file it will create a *table* for storing the comments as *rows*. Each row will have three *columns* (id, author and message).

```ruby
require 'data_mapper'

DataMapper::Logger.new($stdout, :debug)
DataMapper.setup(:default, 'sqlite:my-database.db')

class Comment
  include DataMapper::Resource

  property :id, Serial
  property :author, String
  property :message, Text
end

DataMapper.finalize
DataMapper.auto_upgrade!
```

When you restart your application after this change, it should print a "CREATE TABLE" line to your terminal and create a `my-database.db` file in the current directory.

Use [DB Browser for SQLite][sqlitebrowser] to open the `my-database.db` database. Have a look at the database structure and find the table and columns which the application created. The table doesn't yet contain any data, but we'll solve that next.

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/7ecc3c334c1638739e715723bb713d2bb1ac299f)


### Writing to the database

In the `/add-comment` route, use the following code to save the comment to the database.

```ruby
Comment.create(
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
)
```

Go add some comments to the guestbook. Then use DB Browser to browse the data in the `comments` table and check that the comments you just wrote were added there.

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/4ecec04e19f14b9aa9db4707bd6e16fccf85cf5c)


### Reading from the database

In the `/guestbook.html` route, use the following code to get a list of all the comments in the database. After switching to use that, you can remove the `$comments` variable.

```ruby
Comment.all(:order => [:date.desc])
```

Check what the guestbook page looks like now. It should contain all the comments you wrote earlier, and the newest comment should be on top.

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/58e00d040dd690349a86b1bd1e0fb84688bd1a45)


## Calculating the number of comments using a database

Now that all the data is in the database, we have more power at hand for querying that data. For example we can easily count how many comments there are in total and how many comments were added during the last 15 minutes. If we later implement commenting for photos, likewise we could easily find the photos with the most commenting activity.

Use `Comment.count` to get the total number of comments, and show it on the guestbook page.

Use `Comment.count(:date.gt => Time.now - (15 * 60))` to get the number of comments which are newer than 15 minutes, and show it on the guestbook page.

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/24d24573e294081346063722869b2334abbe3157)


[web-form]: https://en.wikipedia.org/wiki/Form_(HTML)
[html-form]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form
[html-input]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input
[html-button]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button
[http-methods]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods
[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[ruby-datetime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html
[ruby-strftime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html#method-i-strftime
[ruby-array]: http://docs.ruby-lang.org/en/2.2.0/Array.html
[sqlitebrowser]: http://sqlitebrowser.org/
