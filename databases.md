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

[View solution](https://github.com/orfjackal/web-intro-project/commit/eae9445e5fedf66b0638190b87897162220533b7)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/227a4d4f674068769bde587ec7989899fc65a853)


## Keep comments in a text file

Let's keep things simple and implement as much as we can without a database. That way you will also see how a database makes many things better. At first we'll just save the comments into a text file.


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/88e4d40a1f4989c8c22a0f2b58585b6d322c05c2)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/393045ef7db701d3781860cdd541f7b15b38ae61)


## Keep comments in application memory

Though the comments are now visible, they are very limited. For example we cannot easily edit or remove comments, and using templates for them is hard, because the comments are in an unstructured text file.

To solve these issues, we will store the comments in application memory, in a structured data structure. Because the comments are only in application memory, they will disappear when the application is restarted, but the code will be one step away from switching to use a real database which will save the data to the disk in a structured format.

Create a global variable `$comments` which starts as an empty list `[]` (in Ruby lists are called [arrays][ruby-array]). In the `/add-comment` route append new comments to the `$comments` list using `<<`. In the `/pictures/:picture.html` route find the comments of the current picture from among all the comments in `$comments` and give them to the template as `@comments`.

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

[View solution](https://github.com/orfjackal/web-intro-project/commit/42b309bf24fbb6966a2d1ea07e6313872b127113)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/a83781af7cd68fc9dc8da148c4b58ef904407e36)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/cf8a06ec3891f32303f13f1297b61825224a24ac)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/5f9f17e071bef0537d92e68f778df9e986a2910e)


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

[View solution](https://github.com/orfjackal/web-intro-project/commit/2e308db43863c2ac4d63d48e4618b37dbd0c7dcf)


## Count the comments using a database

Now that all the data is in the database, we have more power at hand for querying that data. For example we can easily count how many comments there are for each picture. The DataMapper library provides methods for that and other simple things, but if you'll learn to write [SQL][sql], you'll be able to do even very complex queries.


### Comment count styles

Let's start with just a placeholder for the number of comments, in order to first determine what it should look like.

In `views/pictures.erb`, add the text "0 comments" on a line below the picture, and wrap it and the picture into a `<div>` element, so that they will be grouped together.

```erb
<% for url in @picture_urls %>
<div class="album-caption">
<a href="<%= url.sub(/\.\w+$/, '.html') %>"><img class="album-photo" src="<%= url %>"></a>
<br>0 comments
</div>
<% end %>
```

Use some CSS to make it look nice.

```css
.album-caption {
    float: left;
    text-align: center;
    padding-bottom: 0.5em;
    font-size: 80%;
}
```

![Comment count placeholders with style](comment-count-styles.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/29715d3d7ec4a6a27923f780bf07da9acca4fa81)


### Comment counts

Next replace the placeholder with the actual number of comments. The `Comment.count` method will return the number of comments in the database. You can also use it to count only those comments that [satisfy a particular criteria][datamapper-find], as in the following code.

It used to be enough to give the `pictures` template just the list of picture URLs, but now that there are more pieces of data related to each picture, it's better to give the template a list of hashes. This way the program logic will stay out of the templates, making the code more maintainable.

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
<% for picture in @pictures %>
<div class="album-caption">
<a href="<%= picture[:page_url] %>"><img class="album-photo" src="<%= picture[:picture_url] %>"></a>
<br><%= picture[:comments] %> comments
</div>
<% end %>
```

Add some comments for one picture and check that its comment count increases, but the others stay unchanged.

![Actual comment counts](comment-counts.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/f220bc251f69e4e15eb07fdfaee57f7b1749b2fe)


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
[sql]: https://www.techopedia.com/definition/1245/structured-query-language-sql
[sqlite]: https://www.sqlite.org/
[datamapper]: http://datamapper.org/
[datamapper-find]: http://datamapper.org/docs/find.html
[sqlitebrowser]: http://sqlitebrowser.org/
