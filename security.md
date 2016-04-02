---
layout: chapter
title: Security
permalink: /security/
next: /the-end/
---

Before making your web applications available for others to use, you should learn the basics of how to write secure applications, or else you and the visitors of your site will be vulnerable to hackers.


## Injection attacks

Try adding a comment to your site with the author `<h1>Evil Hacker</h1>` and the message `<script>alert("You've been hacked!");</script>`. You should see a popup window and the comments should show the author in big letters.

![Hacked commenting feature](comments-hacked.png)

This is an example of a classical web application security flaw. The application takes input from the user and puts it as-is into a context where it can be interpreted as code. In the web browser the attack is called [cross-site scripting (XSS)][xss] and in the database it is called [SQL injection][sql-injection].

[![xkcd cartoon about SQL injection](xkcd-sql-injection.png)][xkcd-sql-injection]


## How to avoid

The solution to injection attacks is to always sanitize your input data. For example in HTML the `<` character needs to be replaced with `&lt;` which the web browser will then show as &lt; instead of interpreting it as the start of an HTML tag.


### Escape explicitly

A common *but inferior* way is to explicitly escape the data at every place where it could be dangerous. For example in Sinatra you can create a [helper method for calling escape_html][sinatra-escape-html], which you would then use in your templates like this:

```erb
<%= h scary_output %>
```

The problem with this approach is that if you forget it in even one place, your application will have a security hole.

### Escape by default

A better way is to make the way of least effort also the secure way. In Sinatra you can [escape all HTML automatically][sinatra-auto-escape-html], so that every `<%= scary_output %>` is secure by default. For those cases where you actually need to inject HTML into a template, you can use the special form `<%== html_on_purpose %>` which won't be escaped.

In `app.rb`, add the following code to the beginning of the file to enable HTML escaping by default.

```ruby
require 'erubis'

set :erb, :escape_html => true
```

Have a look at the hacked picture page now. A bit too much was escaped, so in `views/layout.erb` you will need to disable escaping for the page content; replace `<%=` with `<%==`.

```erb
<%== yield %>
```

Now your web site is safe from the XSS attack.

![Secured commenting feature](comments-secured.png)

[View solution](https://github.com/orfjackal/web-intro-project/commit/9b126d0b86349561d17dc75330808bf515b1d608)


## OWASP Top 10

The [OWASP Top 10 Project][owasp-top10] lists the ten most critical web application security risks and how to avoid them. The list has stayed mostly the same from year to year, so the industry as a whole has not learned to avoid these issues. If you learn them already now, before getting into the industry, maybe some day drops of water will make an ocean.


[xss]: https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)
[sql-injection]: https://www.owasp.org/index.php/SQL_Injection
[xkcd-sql-injection]: https://xkcd.com/327/
[sinatra-escape-html]: http://www.sinatrarb.com/faq.html#escape_html
[sinatra-auto-escape-html]: http://www.sinatrarb.com/faq.html#auto_escape_html
[owasp-top10]: https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project
