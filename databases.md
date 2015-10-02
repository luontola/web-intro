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

TODO

- render comments using templates, will need set the [layout option][sinatra-templates] to `false` because otherwise Sinatra will try to wrap the template in `layout.rb`
- [DateTime][ruby-datetime] can be rendered as text using [strftime][ruby-strftime], for example `date.strftime('%Y-%m-%d %H:%M')`

```ruby
comment = erb :comment, :layout => false, :locals => {
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
}
```

![Comments rendered using templates](/screenshots/comments-html.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/0098815dfb61eae146992e1dd33e45b552ad9255)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Storing comments in memory

TODO

- now cannot easily edit or remove comments, or change the representation of old comments
- switch to using in-memory data structure; doesn't persist between restarts, but lets us create the templates and interfaces
- create an [array][ruby-array] for comments as a *global variable* (in Ruby signified by the `$` prefix), put this code outside the routes:

```ruby
$comments = []
```

- comment hashes can be appended to the `$comments` array like this:

```ruby
$comments << {
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
}
```

- update the template `guestbook.rb` to render all the comments similar to we rendered all the pictures in `pictures.erb`
  - will need to access the comment hash elements similar to how we accessed `PAGES`
- check that adding comments works
- now it will be easy to change the templates or edit/remove old comments
- try restarting the server to see that the comments are lost when you restart it; a database is needed now

[View solution](https://github.com/orfjackal/web-intro-project/commit/243a1734f0b048eea92efc702a56767d88263034)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Storing comments in a database

TODO


### Creating the database

TODO

- (optional) download [SQLite](https://www.sqlite.org/), unpack the `sqlite3` executable somewhere on PATH, try to run command `sqlite3 test.db`
- install [DataMapper and its dm-sqlite-adapter](http://datamapper.org/getting-started.html)

```
gem install data_mapper dm-sqlite-adapter
```

- create the database table using DataMapper, add this code to the beginning of your app

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

- start your app, check that console contains "CREATE TABLE" and that `my-database.db` file was created to the current directory
- you can reset the database by removing `my-database.db` or by calling the method `DataMapper.auto_migrate!`

[View solution](https://github.com/orfjackal/web-intro-project/commit/7ecc3c334c1638739e715723bb713d2bb1ac299f)

- install [DB Browser for SQLite][sqlitebrowser]
- open the `my-database.db` file in DB Browser, see that it contains a `comments` table with the fields we specified using DataMapper

TODO: screenshot


### Writing to the database

TODO

- save comment to database using the following code

```ruby
Comment.create(
  :name => params['name'],
  :comment => params['comment'],
  :date => DateTime.now
)
```

- check with DB Browser that the data was added

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/4ecec04e19f14b9aa9db4707bd6e16fccf85cf5c)


### Reading from the database

TODO

- read comments from database using the following code
- remove `$comments`

```ruby
Comment.all(:order => [:date.desc])
```

- see results, newest should be at top

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/58e00d040dd690349a86b1bd1e0fb84688bd1a45)


## Calculating the number of comments using a database

TODO

- database gives some extra power in handling the data
- calculate total number of comments (for example for another page)
- `Comment.count` returns the number of comments
- `Comment.count(:date.gt => Time.now - (15 * 60)` returns the number of comments which are newer than 15 minutes

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
