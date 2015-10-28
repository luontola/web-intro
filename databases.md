---
layout: chapter
title: Databases
permalink: /databases/
next: /security/
---

Most applications store their data in a database. Think of databases as Excel sheets with possibly millions of rows of data, but with more powerful tools for manipulating and summarizing the data. In this chapter we will use databases to implement a commenting feature for the pictures on your site.


## Web forms

Web applications can receive input from the user though [web forms][web-form]. A web form is marked by a [`<form>`][html-form] element which can contain for example [`<input>`][html-input] text fields for the user to write into and a [`<button>`][html-button] for submitting the form.


### Comment form

In `views/picture.erb`, use the following HTML to create a form.

```erb
<h2>Comments</h2>

<form action="/add-comment" method="post" class="add-comment">
    <input type="hidden" name="picture" value="<%= @picture %>">
    <label for="author">Name</label>
    <br><input type="text" name="author">
    <br><label for="message">Comment</label>
    <br><textarea type="text" name="message"></textarea>
    <br><button type="submit">Add Comment</button>
</form>
```

This form has a hidden field for associating the submitted comment with a particular picture.

Change the `/pictures/:picture.html` tell the above template the name of the picture as `@picture`.

```ruby
get '/pictures/:picture.html' do
  @picture = params['picture']
  @picture_url = find_picture_url(params['picture']) or halt 404
  erb :picture
end
```

In `public/style.css`, you can set the size of the form elements. The following code uses [CSS selector combinators][css-selectors] to limit the custom styles inside the comment form, to avoid accidentally changing the style of unrelated form elements.

```css
.add-comment input, .add-comment textarea {
    width: 20em;
}

.add-comment textarea {
    height: 3em;
}
```

Check that the form looks good.

![Form for writing comments](comments-form.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/792421d12596b9ee5cbb3b1b9817b135aa912cb4)


### Receive form parameters

Write something into the form and press the submit button. You should get an error page because the necessary route is missing. Add the following route to your application and try to submit the form again. It should now work.

```ruby
post '/add-comment' do
  puts params
  redirect '/pictures/' + params['picture'] + '.html'
end
```

Did you notice that the `<form>` element had a `method="post"` attribute and that the route declaration also mentions `post`? This refers to the [HTTP request methods][http-methods] of which the most common ones are GET and POST. GET is meant for reading pages and it should not change the application's state, so you can safely do multiple GET request to a page. POST is used for sending data to the web server and it may be used to change the application's state.

Try again writing something to the form and submit it. The `puts` method will print the parameters to the terminal where your application is running. It should print something like `{"picture"=>"annoyed-cat", "author"=>"Ruby", "message"=>"How cute~~!"}`. After submitting the form, your web browser should end up on the same picture page where it was.

[View solution](https://github.com/orfjackal/web-intro-project/commit/ef1d973ce1752aaa54e52c7595ec57253e84b7b9)


## Keep comments in a text file

Let's keep things simple and implement as much as we can without a database. That way you will also see how a database makes many things better. At first we'll just save the comments into a text file.


### Write comments to text file

Use the following code in your `/add-comment` route to append the comment and the name of its author to a text file.

```ruby
post '/add-comment' do
  File.open('comments_' + params['picture'] + '.txt', 'a') do |f|
    f.puts(params['author'] + ': ' + params['message'])
  end
  redirect '/pictures/' + params['picture'] + '.html'
end
```

Try to submit some comments to the guestbook and check that they are added to `comments_*.txt`.

[View solution](https://github.com/orfjackal/web-intro-project/commit/d75f43d80b9d5fa09c7c82d6618722dce08c7493)


### Read comments from text file

Use the following code to read the contents of the text file and give it as a parameter to the picture template. The `if File.exist?` avoids the program crashing when the text file doesn't yet exist.

```ruby
get '/pictures/:picture.html' do
  @picture = params['picture']
  @picture_url = find_picture_url(params['picture']) or halt 404
  @comments = IO.read('comments_' + params['picture'] + '.txt') if File.exist?('comments_' + params['picture'] + '.txt')
  erb :picture
end
```

In `views/picture.erb`, add the following code to show the comments.

```erb
<p><%= @comments %></p>
```

![Show comments, bare bones solution](comments-txt.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/fe7f495e4a86ce92deae71170f6ced09307c2936)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## In-memory database

Though the comments are now visible, they are very limited. For example we cannot easily edit or remove comments, and using templates for them is hard, because the comments are in a simple text file.

To solve these issues, we will store the comments in application memory in an easier to manage data structure. Because the comments are only in application memory, they will disappear when the application is restarted, but the code will be one step away from switching to use a real database.

Create an empty [array][ruby-array] with `[]` when the program starts and store it in the `$comments` variable. In the `/add-comment` route append new comments to `$comments`. In the `/pictures/:picture.html` route read the comments from `$comments` and give them to the template.

TODO: page title

```ruby
$comments = []

get '/pictures/:picture.html' do
  @picture = params['picture']
  @picture_url = find_picture_url(params['picture']) or halt 404
  @comments = $comments.select { |comment| comment[:picture] == params['picture'] }
  erb :picture
end

post '/add-comment' do
  $comments << {
    :picture => params['picture'],
    :author => params['author'],
    :message => params['message'],
    :added => DateTime.now
  }
  redirect '/pictures/' + params['picture'] + '.html'
end
```

Try adding some comments. They should look like a data structure. Try restarting the application and notice how all comments disappear then.

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/a1cf1ba00f63c424c0938a52f3bf4c2d345dc022)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Comment template

TODO: rewrite this section

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

![Comments rendered using templates](comments-html.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/0d8152da3c54ad0eb26acf8e8b418c766b084183)


## Keep comments in a database

The commenting feature is now working nicely and the code is easy to change, but when you restart the server (try it), all the comments will be lost. To keep the data safe and the code maintainable, a database is needed.

We will use the [SQLite](https://www.sqlite.org/) database, the [DataMapper](http://datamapper.org/) library for accessing SQLite in Ruby, and [DB Browser for SQLite](http://sqlitebrowser.org/) as a UI for seeing inside the database. If you haven't yet installed them, go through the [installation guide](/install/) and then come then back here.


### Create a database for comments

Add the following code to the beginning of your application. It will create a `my-database.db` file where all your data will be saved. In that file it will create a *table* for storing the comments as *rows*. The table will have four *columns* (id, name, comment and date).

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

![The database structure which was created](database-structure.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/8f3ddb5dc9299228d77ea993b0db8d9abafcfffa)


### Write comments to database

In the `/add-comment` route, where you already append the comment to `$comments`, add the following code to save the comment also to the database.

```ruby
Comment.create(
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
)
```

Go add some comments to the guestbook. Then use DB Browser to browse the data in the `comments` table and check that the comments you just wrote were added there.

![Some data inside the database](database-data.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/341d2be9c1503cd268992755b8d6cdd2d2638f00)


### Read comments from database

In the `/guestbook.html` route, replace `$comments` with the following code which will read all comments from the database (ordered newest first). After that change, you can remove the `$comments` variable and all code that still uses it.

```ruby
Comment.all(:order => [:date.desc])
```

Check what the guestbook page looks like now. It should contain all the comments you wrote earlier, and the newest comment should be on top.

![Showing comments from the database](comments-database.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/3fbf2743e41e240f768872f99df318c8201fcf51)


## Calculating the number of comments using a database

TODO: rewrite this section

Now that all the data is in the database, we have more power at hand for querying that data. For example we can easily count how many comments there are in total and how many comments were added during the last 15 minutes. If we later implement commenting for photos, likewise we could easily find the photos with the most commenting activity.

Use `Comment.count` to get the total number of comments, and show it on the guestbook page.

Use `Comment.count(:date.gt => Time.now - (15 * 60))` to get the number of comments which are newer than 15 minutes, and show it on the guestbook page.

![Showing the number of comments](comments-counts.png)


### Comment count styles

TODO

[View solution](https://github.com/orfjackal/web-intro-project/commit/fdc2eb9ce33bbb42ff1b10611dc4e938cc67867c)


### Comment counts

TODO

[View solution](https://github.com/orfjackal/web-intro-project/commit/9fcd6dd2b26e04f978e9fab658d1dada018c2768)


[web-form]: https://en.wikipedia.org/wiki/Form_(HTML)
[html-form]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form
[html-input]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input
[html-button]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button
[css-selectors]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started/Selectors
[http-methods]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods
[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[ruby-datetime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html
[ruby-strftime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html#method-i-strftime
[ruby-array]: http://docs.ruby-lang.org/en/2.2.0/Array.html
[sqlitebrowser]: http://sqlitebrowser.org/
