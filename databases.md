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


### Guestbook page

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

[View solution](https://github.com/orfjackal/web-intro-project/commit/5b8b925bd57218220afcf71c9bfaaa41294e2fe0)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/f69218f410146b31e507e1011f8ffc2898d82bf8)


## Storing comments in a text file

Let's keep things simple and implement as much as we can without a database. That way you will also see how a database makes many things better. At first we'll just save the comments into a text file.


### Writing to the text file

Use the following code in your `/add-comment` route to append the comment and the name of its author to a text file.

```ruby
post '/add-comment' do
  File.open('comments.txt', 'a') do |f|
    f.puts(params['name'] + ': ' + params['comment'])
  end
  redirect '/guestbook.html'
end
```

Try to submit some comments to the guestbook and check that they are added to `comments.txt`.

[View solution](https://github.com/orfjackal/web-intro-project/commit/10c8817bda8dddee2a973c34901450d4f8d40510)


### Reading from the text file

Now let's show the comments on the guestbook page.

Use the following code to read the contents of the text file and give it as a parameter to the guestbook template. The `if File.exist?` avoids the program crashing when the text file doesn't yet exist.

```ruby
get '/guestbook.html' do
  comments = IO.read('comments.txt') if File.exist?('comments.txt')
  render_page :guestbook, {:comments => comments}
end
```

Then in `guestbook.erb` add the following code to show the comments.

```html
<p><%= comments %></p>
```

![Show comments, bare bones solution](/screenshots/comments-txt.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/f8a760541b7673f2e25da31a7ca8d6aea8c0f54d)

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

Then in your `/add-comment` route use `erb` to rendered HTML using the `comment.erb` template and append the result to the `comments.txt` file.

```ruby
post '/add-comment' do
  File.open('comments.txt', 'a') do |f|
    comment = erb :comment, :layout => false, :locals => {
      :name => params['name'],
      :comment => params['comment'],
      :date => DateTime.now
    }
    f.puts(comment)
  end
  redirect '/guestbook.html'
end
```

The [layout option][sinatra-templates] had to be set to `false` because otherwise Sinatra will try to wrap the template in `layout.rb`.

Try adding some comments and write some CSS to make it look the way you like it. If you want the template to render the [DateTime][ruby-datetime] in a different format, see the documentation for [strftime][ruby-strftime].

![Comments rendered using templates](/screenshots/comments-html.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/55162e260f369deeceba6926c979a948db2af2cf)


## Storing comments in memory

Though the comments are now visible, they are very limited. For example we cannot easily edit or remove comments, or change the template for existing comments.

As a step towards using a database, we will store the comments in application memory. That way the comments will disappear when the application is stopped, but we'll be able do the necessary changes to our templates. After that switching to a real database will be a small step.

Create an empty [array][ruby-array] (`[]`) when the program starts and store it in the `$comments` variable. In the `/add-comment` route append new comments to `$comments`. In the `/guestbook.html` route read the comments from `$comments` and give them to the guestbook template.

```ruby
$comments = []

get '/guestbook.html' do
  render_page :guestbook, {:comments => $comments}
end

post '/add-comment' do
  $comments << {
    :name => params['name'],
    :comment => params['comment'],
    :date => DateTime.now
  }
  redirect '/guestbook.html'
end
```

Finally update the template `guestbook.rb` to render all the comments similar to how we rendered all the pictures in `pictures.erb`.

```html
<% for comment in comments %>
<p>
  <span class="comment-author"><%= comment[:name] %> wrote on <%= comment[:date].strftime('%Y-%m-%d %H:%M') %>:</span>
  <br><span class="comment-message"><%= comment[:comment] %></span>
</p>
<% end %>
```

Check that adding comments works.

[View solution](https://github.com/orfjackal/web-intro-project/commit/6872f8f483706fc7dd1b3184d55a51550986b72b)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Storing comments in a database

The commenting feature is now working nicely and the code is easy to change, but when you restart the server (try it), all the comments will be lost. To keep the data safe and the code maintainable, a database is needed.

We will use the [SQLite](https://www.sqlite.org/) database, the [DataMapper](http://datamapper.org/) library for accessing SQLite in Ruby, and [DB Browser for SQLite](http://sqlitebrowser.org/) as a UI for seeing inside the database. If you haven't yet installed them, go through the [installation guide](/installation/) and then come then back here.


### Creating the database

Add the following code to the beginning of your application. It will create a file `my-database.db` where all your data will be saved. In that file it will create a *table* for storing the comments as *rows*. Each row will have three *columns* (id, author and message).

```ruby
require 'data_mapper'

DataMapper::Logger.new($stdout, :debug)
DataMapper.setup(:default, 'sqlite:my-database.db')

class Comment
  include DataMapper::Resource

  property :id, Serial
  property :name, String
  property :comment, Text
  property :date, DateTime
end

DataMapper.finalize
DataMapper.auto_upgrade!
```

When you restart your application after this change, it should print a "CREATE TABLE" line to your terminal and create a `my-database.db` file in your project folder.

Use [DB Browser for SQLite][sqlitebrowser] to open the `my-database.db` database. Have a look at the database structure and find the table and columns which the application created. The table doesn't yet contain any data, but we'll solve that next.

![The database structure which was created](/screenshots/database-structure.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/11d7673c40fb7da0a39271ea69b340effda7d7e2)


### Writing to the database

In the `/add-comment` route, where you already append the comment to `$comments`, add the following code to save the comment also to the database.

```ruby
Comment.create(
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
)
```

Go add some comments to the guestbook. Then use DB Browser to browse the data in the `comments` table and check that the comments you just wrote were added there.

![Some data inside the database](/screenshots/database-data.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/9b4602dfe6f0967ac990baf8c9e18a6e8c32a11d)


### Reading from the database

In the `/guestbook.html` route, replace `$comments` with the following code which will read all comments from the database (ordered newest first). After that change, you can remove the `$comments` variable and all code that still uses it.

```ruby
Comment.all(:order => [:date.desc])
```

Check what the guestbook page looks like now. It should contain all the comments you wrote earlier, and the newest comment should be on top.

![Showing comments from the database](/screenshots/database-guestbook.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/19a89f7ebd45db0ea15e760cc05980fe8cce6a49)


## Calculating the number of comments using a database

Now that all the data is in the database, we have more power at hand for querying that data. For example we can easily count how many comments there are in total and how many comments were added during the last 15 minutes. If we later implement commenting for photos, likewise we could easily find the photos with the most commenting activity.

Use `Comment.count` to get the total number of comments, and show it on the guestbook page.

Use `Comment.count(:date.gt => Time.now - (15 * 60))` to get the number of comments which are newer than 15 minutes, and show it on the guestbook page.

![Showing the number of comments](/screenshots/database-counts.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/0b540c4b63eb562340f07a033e325f180fb1684d)


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
