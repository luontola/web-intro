---
layout: chapter
title: Installation Guide
permalink: /install/
---

## 1. Text editor

Install a programmer's text editor such as [Visual Studio Code](https://code.visualstudio.com/) or [Atom](https://atom.io/).


## 2. Ruby

Install the Ruby programming language

* On Windows, install [Ruby **2.1** using RubyInstaller](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.1.7.exe)
  * In the installer, select the checkbox "Add Ruby executables to your PATH"
  * Ruby 2.2 is not yet supported by *dm-sqlite-adapter* (because *do_sqlite3-0.10.16* does not include the binary for it), so **if you are using Windows you must use Ruby 2.1 or earlier**

<hr>

* On OS X, install [Homebrew](http://brew.sh/) by following the instructions on [its homepage](http://brew.sh/), and then, in your [terminal][terminal], run the command `brew install ruby`, and **after installing Ruby, be sure to restart the terminal app**.
  * When it is asking for your password, write the password you use for signing in to your computer. **Note!** Typing your password will not show up as any characters in the prompt.

<hr>

* On other operating systems, see [Installing Ruby](https://www.ruby-lang.org/en/documentation/installation/)


## 3. SQLite

* On Windows, download the [sqlite-dll package](http://www.sqlite.org/2015/sqlite-dll-win32-x86-3081101.zip) and copy the `sqlite3.dll` file to Ruby's bin folder (`C:\Ruby21\bin`)


## 4. DB Browser for SQLite

* Install [DB Browser for SQLite](http://sqlitebrowser.org/)


## 5. Ruby libraries

* Run the following command in your [terminal][terminal] (known as the [Command Prompt][winprompt] on Windows) to install some Ruby libraries which we'll use in this tutorial:  
  `gem install sinatra:1.4.7 sinatra-contrib:1.4.7 data_mapper dm-sqlite-adapter erubis --no-ri --no-rdoc`
  * While the computer is in the middle of performing the command, a cursor will blink on an empty line. When the command is completed, there will be stuff in front of the blinking cursor.
  * On OS X and Linux, if the command fails complaining about permissions, add the `sudo` word to the beginning of the command and try again.


## 6. Test that it all works

* Save [this test.rb file](/test.rb) in a folder you can locate (instructions for getting the full path for a folder: [Windows][winpath] and [Mac/Linux][macpath]), run the command `ruby test.rb` in your [terminal][terminal], either by typing it or copy-pasting it in.
  * Ensure you have navigated into the same folder (see instructions for [Windows][winnavigation] and [Mac/Linux][macnavigation]) as where you saved the test.rb file.
* If the [terminal][terminal] says "`Database OK`" and you can visit <http://localhost:4567/> with your web browser (e.g. [Firefox](https://www.mozilla.org/en-US/firefox/new/), Safari, or similar) and if it says "`Web server OK`", then you're all done!


### Workarounds to some problems

#### getaddrinfo: The requested name is valid, but no data of the requested type was found. (SocketError)

If you get this error when running `ruby test.rb`, it means that the web server is failing to listen on the `0.0.0.0` host, your computer's public network interface. A workaround is to make it listen only private connections.

Try running `ruby test.rb -o 127.0.0.1` and if it works, you will need to add the `-o 127.0.0.1` option also when running your application during the rest of this guide.


[terminal]: http://askubuntu.com/questions/38162/what-is-a-terminal-and-how-do-i-open-and-use-it
[macnavigation]: http://askubuntu.com/questions/232442/how-do-i-navigate-between-directories-in-terminal
[winnavigation]: http://www.pcstats.com/articleview.cfm?articleid=1723&page=3
[winprompt]: https://redmondmag.com/articles/2014/11/14/windows-10-command-prompt.aspx
[winpath]: http://www.tomshardware.co.uk/forum/252517-44-full-path-file-folder-windows-folders
[macpath]: http://josharcher.uk/code/find-path-to-folder-on-mac/
