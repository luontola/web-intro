---
layout: chapter
title: Databases
permalink: /databases/
next: /security/
---

TODO: explain what databases are

- we implement a commenting feature (later could be expanded to comments per picture; left as an exercise for the reader)


## Web forms

- create guestbook page and a form for adding comments

![Form for writing comments](/screenshots/comments-form.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/796018db7c92e03d10b72a453632a95743d4e572)

- try to submit, see error message and suggestion from Sinatra, go add that route

TODO: explain forms and POST

- print parameters (`puts params`) to see what it contains and redirect to the guestbook page
- try to submit, should print something like `{"name"=>"Ruby", "comment"=>"This is fun!"}`

[View solution](https://github.com/orfjackal/web-intro-project/commit/b4e967b6399704cf96ad541c231ebb6b7dac2aa3)


## Storing comments in a text file

- append comments to a text file, check that lines are added to it when you submit the form

```ruby
File.open('comments.txt', 'a') do |f|
  f.puts(params['name'] + ': ' + params['comment'])
end
```

[View solution](https://github.com/orfjackal/web-intro-project/commit/93838d36bd66d22a3d6363abfb49dba0ef888498)

- show comments on page; the following code can be used to read the contents of the file, so that it works also when the file doesn't yet exist

```ruby
comments = IO.read('comments.txt') if File.exist?('comments.txt')
```

![Show comments, bare bones solution](/screenshots/comments-txt.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/1db02859480c0a2bccfcf748d9380ef66d3803a9)

*Note: The example solution has a security vulnerability, but we'll address that in a [later chapter](/security/).*


## Rendering comments using templates

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

### Creating the database

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

- read comments from database using the following code
- remove `$comments`

```ruby
Comment.all(:order => [:date.desc])
```

- see results, newest should be at top

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/58e00d040dd690349a86b1bd1e0fb84688bd1a45)


## Calculating the number of comments using a database

- database gives some extra power in handling the data
- calculate total number of comments (for example for another page)
- `Comment.count` returns the number of comments
- `Comment.count(:date.gt => Time.now - (15 * 60)` returns the number of comments which are newer than 15 minutes

TODO: screenshot

[View solution](https://github.com/orfjackal/web-intro-project/commit/24d24573e294081346063722869b2334abbe3157)


[sinatra-templates]: http://www.sinatrarb.com/intro.html#Views%20/%20Templates
[ruby-datetime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html
[ruby-strftime]: http://docs.ruby-lang.org/en/2.2.0/DateTime.html#method-i-strftime
[ruby-array]: http://docs.ruby-lang.org/en/2.2.0/Array.html
[sqlitebrowser]: http://sqlitebrowser.org/
