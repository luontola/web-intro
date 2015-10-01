---
layout: chapter
title: Installation
permalink: /installation/
next: /templating/
---


## Install Ruby

To build apps and other things with Ruby, we need to setup some software and the developer environment for your computer.

Follow the instructions for your operating system. If you run into any problems, don&#8217;t panic. Inform us at the event and we can solve it together.

* [Setup for OS X](#setup-for-os-x)
* [Setup for Windows](#setup-for-windows)
* [Setup for Linux](#setup-for-linux)
* [Alternative Installment for all OS](#virtual-machine)
* [Using a Cloud Service - No Installation Required](#using-a-cloud-service)

<hr />

## Setup for OS X

### *1.* Let&#8217;s check the version of the operating system.

Click the Apple menu and choose *About this Mac*.

![Apple menu](/screenshots/apple-menu.png "Apple menu")

### *2.* In the window you will find the version of your operating system.
If your version number starts with 10.6, 10.7, 10.8, 10.9 or 10.10 this guide is for you. If it&#8217;s something else, we can setup your machine at the event.

![About this Mac dialog](/screenshots/apple-dialog.png "About this Mac dialog")

### *3a.* If your OS X version is 10.9 or higher:

If your version number starts with 10.9 or 10.10, follow these steps. We are installing homebrew and rbenv.

#### *3a1.* Install Command line tools on terminal:

{% highlight sh %}
xcode-select --install
{% endhighlight %}

#### *3a2.* Install [Homebrew](http://brew.sh/):

{% highlight sh %}
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

#### *3a3.* Install [rbenv](https://github.com/sstephenson/rbenv):

{% highlight sh %}
brew update
brew install rbenv rbenv-gem-rehash ruby-build
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
echo 'export PATH="$HOME/.rbenv/shims:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
{% endhighlight %}

#### *3a4.* Build Ruby with rbenv:

You can find the newest version of Ruby with the command "rbenv install -l".

{% highlight sh %}
rbenv install 2.2.3
{% endhighlight %}

If you got "OpenSSL::SSL::SSLError: ... : certificate verify failed" error, try it this way.

{% highlight sh %}
brew install curl-ca-bundle
cp /usr/local/opt/curl-ca-bundle/share/ca-bundle.crt `ruby -ropenssl -e 'puts OpenSSL::X509::DEFAULT_CERT_FILE'`
{% endhighlight %}

#### *3a5.* Set default Ruby:

{% highlight sh %}
rbenv global 2.2.3
{% endhighlight %}

## Setup for Windows

### *1.* Install Ruby

Download [RailsInstaller](https://s3.amazonaws.com/railsinstaller/Windows/railsinstaller-3.1.0.exe) and run it. Click through the installer using the default options.

Open `Command Prompt with Ruby on Rails` and run the following command:

{% highlight sh %}
ruby -v
{% endhighlight %}

It will check that your Ruby is successfuly installed. If everything is ok, you will see your Ruby version (should be 2.1).

## Possible errors

### Gem::RemoteFetcher error

If you get this error when running `rails new railsgirls` or `gem update rails`:

{% highlight sh %}
Gem::RemoteFetcher::FetchError: SSL_connect returned=1 errno=0 state=SSLv3 read
server certificate B: certificate verify failed (https://rubygems.org/gems/i18n-
0.6.11.gem)
{% endhighlight %}

This means you have an older version of Rubygems and will need to update it manually first verify your Rubygems version

{% highlight sh %}
gem -v
{% endhighlight %}

If it is lower than `2.2.3` you will need to manually update it:


First download the [ruby-gems-update gem](https://github.com/rubygems/rubygems/releases/download/v2.2.3/rubygems-update-2.2.3.gem). Move the file to `c:\\rubygems-update-2.2.3.gem` then run:

{% highlight sh %}
gem install --local c:\\rubygems-update-2.2.3.gem
update_rubygems --no-ri --no-rdoc
gem uninstall rubygems-update -x
{% endhighlight %}

Check your version of rubygems

{% highlight sh %}
gem -v
{% endhighlight %}

Make sure it is higher than `2.2.3`. Re-run the command that was failing previously.


### 'x64_mingw' is not a valid platform` Error

Sometimes you get the following error when running `rails server`:
`'x64_mingw' is not a valid platform` If you experience this error after using the RailsInstaller you have to do a small edit to the file `Gemfile`:

Look at the bottom of the file. You will probably see something like this as one of the last lines in the file:
`gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw]`. If you have this line with `:x64_mingw`, then please delete the `:x64_mingw` part. In the end it should just say:
`'tzinfo-data', platforms: [:mingw, :mswin]`

After you did that, please use your Command Prompt again and type `bundle update`.

### *2.* Install a text editor to edit code files

For the workshop we recommend the text editor Atom.

* [Download Atom and install it](https://github.com/atom/atom/releases/latest)
  * Download an atom zip file for windows and decompress it.
  * Copy the folder into your Program Files.
  * Launch atom in the folder.

If you are using Windows Vista or older versions, you can use another editor [Sublime Text 2](http://www.sublimetext.com/2).

Now you should have a working Ruby on Rails programming setup. Congrats!

### *3.* Update your browser

If you use Internet Explorer, we recommend installing [Firefox](mozilla.org/firefox) or [Google Chrome](google.com/chrome).

Open [whatbrowser.org](http://whatbrowser.org) and update your browser if you don't have the latest version.

<hr />

## Setup for Linux

### *1.* Install Ruby

TODO: someone, please write rbenv/ruby installation script for Linux.

### *2.* Install a text editor to edit code files

For the workshop we recommend the text editor Sublime Text.

* [Download Sublime Text and install it](http://www.sublimetext.com/2)


### *3.* Update your browser

Open [whatbrowser.org](http://whatbrowser.org) and update your browser if you don't have the latest version.


Now you should have a working Ruby on Rails programming setup. Congrats!

## Install Sinatra

__COACH__: Explain shortly what [Sinatra](http://www.sinatrarb.com) is.

Remember how we needed to install Ruby on Rails? Similarly we need to install Sinatra:

`gem install sinatra`

### Create your first Sinatra app

Create a `myapp.rb` file with the following contents:

{% highlight ruby %}
require 'sinatra'

get '/' do
  'Hello, world!'
end
{% endhighlight %}

Now, by typing `ruby myapp.rb` in the _Terminal_, you will start your application.
Now you can visit <a href="localhost:4567" target="_blank">localhost:4567</a>. You should
see a ‘Hello, world!’ page, which means that the generation of your new app was successful.
