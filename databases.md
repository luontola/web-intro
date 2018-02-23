---
layout: chapter
title: Databases
permalink: /databases/
next: /security/
---

Most applications store their data in a database. Think of databases as Excel sheets, but with even millions of rows of data and with more powerful tools for manipulating and summarizing the data. In this chapter we will use databases to implement a commenting feature for the pictures on your site.


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
  @title = "Picture"
  @picture = params['picture']
  @picture_url = find_picture_url(params['picture']) or halt 404
  erb :picture
end
```

In `public/style.css`, you can set the size of the form elements. The following code uses [CSS selector combinators][css-selectors] to limit the custom styles inside the comment form, to avoid accidentally changing the style of unrelated form elements. It also uses [relative length units][css-length] which depend on the font size instead of the pixels on the screen.

```css
.add-comment input, .add-comment textarea {
    width: 40ch;
}

.add-comment textarea {
    height: 3em;
}
```

Check that the form looks good.

![Comment form](comment-form.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/6dd52bf61213211b33a872f29d1d73e91c3e898c/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/6dd52bf61213211b33a872f29d1d73e91c3e898c/views/picture.erb">views/picture.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/6dd52bf61213211b33a872f29d1d73e91c3e898c/public/style.css">public/style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/6dd52bf61213211b33a872f29d1d73e91c3e898c">View changes</a>
</aside>


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

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/648b9199f53c5aa0f6205bc3863ae0bec8ffa244/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/648b9199f53c5aa0f6205bc3863ae0bec8ffa244">View changes</a>
</aside>


## Keep comments in a text file

Let's keep things simple and implement as much as we can without a database. That way you will also see how a database makes things better. But we are still making some progress every step of the way and checking our assumptions of what the code does. Working incrementally like this is good practice even in real life.

At first we'll just save the comments into a text file.


### Write comments to a text file

Use the following code in your `/add-comment` route to append the comment and the name of its author to a text file.

```ruby
post '/add-comment' do
  File.open('comments_' + params['picture'] + '.txt', 'a') do |f|
    f.puts(params['author'] + ': ' + params['message'])
  end
  redirect '/pictures/' + params['picture'] + '.html'
end
```

Try adding some comments and check that they are saved to `comments_*.txt`.

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/18c3ef9ec972bbd335280f665914dc7012c5b520/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/18c3ef9ec972bbd335280f665914dc7012c5b520">View changes</a>
</aside>


### Read comments from a text file

Use the following code to read the contents of the text file and pass it to the picture template. The `if File.exist?` avoids the program crashing when the text file doesn't yet exist.

```ruby
get '/pictures/:picture.html' do
  @title = "Picture"
  @picture = params['picture']
  @picture_url = find_picture_url(params['picture']) or halt 404
  comments_file = 'comments_' + params['picture'] + '.txt'
  @comments = IO.read(comments_file) if File.exist?(comments_file)
  erb :picture
end
```

In `views/picture.erb`, add the following code to show the comments.

```erb
<p><%= @comments %></p>
```

*Note: This template has a security vulnerability, but we'll address that in a [later chapter](/security/).*

![Comments raw from a text file](keep-comments-in-a-text-file.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/f7562ccfdeaad267f3c7df0c3326cc78e947a054/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/f7562ccfdeaad267f3c7df0c3326cc78e947a054/views/picture.erb">views/picture.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/f7562ccfdeaad267f3c7df0c3326cc78e947a054">View changes</a>
</aside>


## Keep comments in application memory

Though the comments are now visible, they are very limited. For example we cannot easily edit or remove comments, and using templates for them is hard, because the comments are in an unstructured text file.

To solve these issues, we will store the comments in application memory, in a structured data structure. Because the comments are only in application memory, they will disappear when the application is restarted. But it will be one step closer to saving the data in a real database.

Create a global variable `$comments` which starts as an empty list `[]` (in Ruby lists are also called [arrays][ruby-array]). In the `/add-comment` route append new comments to the `$comments` list using `<<`. In the `/pictures/:picture.html` route find the comments of the current picture from among all the comments in `$comments` and give them to the template as `@comments`.

```ruby
$comments = []

get '/pictures/:picture.html' do
  @title = "Picture"
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

![Comments raw from application memory](keep-comments-in-application-memory.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/25fd412a909f828b30b7ce753962d089a35b0ab5/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/25fd412a909f828b30b7ce753962d089a35b0ab5">View changes</a>
</aside>


## Comment template

In `views/picture.erb`, show the comments in a human-readable format.

```erb
<% for comment in @comments %>
<p><span class="comment-author"><%= comment[:author] %>
  wrote on <%= comment[:added].strftime('%Y-%m-%d %H:%M:%S') %></span>
<br><span class="comment-message"><%= comment[:message] %></span></p>
<% end %>
```

*Note: This template has a security vulnerability, but we'll address that in a [later chapter](/security/).*

Try adding comments and write some CSS to make them look the way you like. If you want the template to render the [DateTime][ruby-datetime] in a different format, see the documentation for [strftime][ruby-strftime].

![Comments rendered using templates](comment-template.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/7e91495c827f4f263b2fe6410c7492ac7c517689/views/picture.erb">views/picture.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/7e91495c827f4f263b2fe6410c7492ac7c517689/public/style.css">public/style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/7e91495c827f4f263b2fe6410c7492ac7c517689">View changes</a>
</aside>


## Keep comments in a database

The commenting feature is now working nicely and the code is easy to change, but when you restart the web server, all the comments will be lost. To keep the data safe and the code maintainable, a database is needed.

We will use the [SQLite][sqlite] database, the [DataMapper][datamapper] library for accessing SQLite in Ruby, and [DB Browser for SQLite][sqlitebrowser] as a GUI for seeing inside the database. If you haven't yet installed them, go through the [installation guide](/install/) and then come then back here.


### Create a database for comments

Add the following code to the beginning of your application. It will create a `my-database.db` file where all your data will be saved. Inside the database it will create a *table* for storing the comments as *rows*. The table will have five *columns* (id, picture, author, message and added).

```ruby
require 'data_mapper'

DataMapper::Logger.new($stdout, :debug)
DataMapper.setup(:default, 'sqlite:my-database.db')

class Comment
  include DataMapper::Resource

  property :id, Serial
  property :picture, String
  property :author, String
  property :message, Text
  property :added, DateTime
end

DataMapper.finalize
DataMapper.auto_upgrade!
```

When you restart your application after this change, it should print a "CREATE TABLE" line to your terminal and create a `my-database.db` file in your project folder.

Use [DB Browser for SQLite][sqlitebrowser] to open the `my-database.db` database. Have a look at the database structure and find the table and columns which the application created. The table doesn't yet contain any data, but we'll solve that next.

![The database structure which the application created](create-a-database-for-comments.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/90ee5d1c9bd9d244895cc8e86ebe9ab73ff99e8f/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/90ee5d1c9bd9d244895cc8e86ebe9ab73ff99e8f">View changes</a>
</aside>


### Write comments to a database

In the `/add-comment` route, use `Comment.create` to save the comment to the database. You can keep the old `$comments` usage still side by side with the new code, so that no existing functionality is broken by the changes.

```ruby
post '/add-comment' do
  Comment.create(
    :picture => params['picture'],
    :author => params['author'],
    :message => params['message'],
    :added => DateTime.now
  )
  $comments << {
    :picture => params['picture'],
    :author => params['author'],
    :message => params['message'],
    :added => DateTime.now
  }
  redirect '/pictures/' + params['picture'] + '.html'
end
```

Go add some comments on your site. Then use DB Browser to browse the data in the `comments` table and check that the comments you just wrote were saved in the database.

![Some comments inside the database](write-comments-to-a-database.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/674dc1e71ae3d1590821befe2fb5a8958a8b2742/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/674dc1e71ae3d1590821befe2fb5a8958a8b2742">View changes</a>
</aside>


### Read comments from a database

In the `/pictures/:picture.html` route, use [`Comment.all`][datamapper-find] to find from the database all comments for that picture, newest first. After this change, you can remove the `$comments` variable and all code that still uses it.

```ruby
get '/pictures/:picture.html' do
  @title = "Picture"
  @picture = params['picture']
  @picture_url = find_picture_url(params['picture']) or halt 404
  @comments = Comment.all(:picture => params['picture'], :order => [:added.asc])
  erb :picture
end
```

Check the comments on the picture page now. It should show the comments which you saw saved in the database.

![Showing comments from the database](read-comments-from-a-database.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/3903ba27cfef5dc73f2c1f6c6f62f973d5830676/app.rb">app.rb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/3903ba27cfef5dc73f2c1f6c6f62f973d5830676">View changes</a>
</aside>


## Count the comments using a database

Now that all the data is in the database, we have more power at hand for querying that data. For example we can easily count how many comments there are for each picture. The DataMapper library provides methods for that and other simple things, but if you'll learn to write [SQL][sql], you'll be able to do even very complex queries.


### Comment count styles

Let's start with just a placeholder for the number of comments, in order to first determine what it should look like.

In `views/pictures.erb`, add the text "0 comments" on a line below the picture, and wrap it and the picture into a `<div>` element, so that they will be grouped together.

```erb
<div class="album">
<% for url in @picture_urls %>
    <div class="album-caption">
        <a href="<%= url.sub(/\.\w+$/, '.html') %>"><img class="album-photo" src="<%= url %>"></a>
        <br>0 comments
    </div>
<% end %>
</div>
```

Use some CSS to make it look nice.

```css
.album {
    display: flex;
    flex-wrap: wrap;
}

.album-caption {
    text-align: center;
    padding-bottom: 0.5em;
    font-size: 80%;
}
```

![Comment count placeholders with style](comment-count-styles.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/29715d3d7ec4a6a27923f780bf07da9acca4fa81/views/pictures.erb">views/pictures.erb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/29715d3d7ec4a6a27923f780bf07da9acca4fa81/public/style.css">public/style.css</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/29715d3d7ec4a6a27923f780bf07da9acca4fa81">View changes</a>
</aside>


### Comment counts

Next replace the placeholder with the actual number of comments. The `Comment.count` method will return the number of comments in the database. You can also use it to count only those comments that [satisfy a particular criteria][datamapper-find], as in the following code.

It used to be enough to give the `pictures` template just the list of picture URLs, but now that there are more pieces of data related to each picture, it's better to give the template a list of [hashes][ruby-hash]. This way the program logic will stay out of the templates, making the code more maintainable.

```ruby
get '/pictures.html' do
  @title = "Lovely Pictures"
  @pictures = picture_urls.map { |url| {
    :picture_url => url,
    :page_url => url.sub(/\.\w+$/, '.html'),
    :comments => Comment.count(:picture => File.basename(url, '.*'))
  }}
  erb :pictures
end
```

The `views/pictures.erb` template needs to be changed to use `@pictures` instead of `@picture_urls`.

```erb
<div class="album">
<% for picture in @pictures %>
    <div class="album-caption">
        <a href="<%= picture[:page_url] %>"><img class="album-photo" src="<%= picture[:picture_url] %>"></a>
        <br><%= picture[:comments] %> comments
    </div>
<% end %>
</div>
```

Add some comments for one picture and check that its comment count increases, but the others stay unchanged.

![Actual comment counts](comment-counts.png)

<aside class="solution">
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/1ccc7b0d35a105960a70c2f6285f128fa45de5ab/app.rb">app.rb</a>
    <a class="file" href="https://github.com/orfjackal/web-intro-project/blob/1ccc7b0d35a105960a70c2f6285f128fa45de5ab/views/pictures.erb">views/pictures.erb</a>
    <a class="diff" href="https://github.com/orfjackal/web-intro-project/commit/1ccc7b0d35a105960a70c2f6285f128fa45de5ab">View changes</a>
</aside>


[web-form]: https://en.wikipedia.org/wiki/Form_(HTML)
[html-form]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form
[html-input]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input
[html-button]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button
[css-selectors]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started/Selectors
[css-length]: https://developer.mozilla.org/en/docs/Web/CSS/length
[http-methods]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods
[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[ruby-datetime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html
[ruby-strftime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html#method-i-strftime
[ruby-array]: http://docs.ruby-lang.org/en/2.2.0/Array.html
[ruby-hash]: http://ruby-doc.org/core-2.2.0/Hash.html
[sql]: http://www.guru99.com/introduction-to-database-sql.html
[sqlite]: https://www.sqlite.org/
[datamapper]: http://datamapper.org/
[datamapper-find]: http://datamapper.org/docs/find.html
[sqlitebrowser]: http://sqlitebrowser.org/
