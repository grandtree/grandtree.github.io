---
layout: post
title:  "Install Jekyll on macOS Monterey"
date:   2021-12-24 16:24:00 +0800
categories: jekyll macOS 
---
I encountered several issues when setting up this blog web site, so it is a good
topic for my first post. 

# Environment

- Version of macOS: Monterey 12.0.1 (21A559)
- HW: MacBook Air (M1, 2020)

# Before something wrong
Per [Jekyll on macOS][jekyll-mac], install xcode command line tools to compile native extensions
```
qy@Yis-MacBook-Air:~$ xcode-select --install
xcode-select: note: install requested for command line developer tools
```

I plan to use pre-installed Ruby because the MacPorts on my machine has not been upgraded
after OS upgrade. 
```
qy@Yis-MacBook-Air:~$ ruby -v
ruby 2.6.8p205 (2021-07-07 revision 67951) [universal.arm64e-darwin21]
```

# Rdoc issue
Try to install bundler and jekyll. Note that I install them to my home directory (--user-install) because I don't need to mess up the pre-installed Ruby environment. 
```
qy@Yis-MacBook-Air:~$ gem install --user-install bundler jekyll
...
Parsing documentation for terminal-table-2.0.0
Installing ri documentation for terminal-table-2.0.0
Parsing documentation for jekyll-4.2.1
Before reporting this, could you check that the file you're documenting
has proper syntax:

  /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/bin/ruby -c lib/jekyll/commands/doctor.rb

RDoc is not a full Ruby parser and will fail when fed invalid ruby programs.

The internal error was:

        (NoMethodError) undefined method `[]' for nil:NilClass

ERROR:  While executing gem ... (NoMethodError)
    undefined method `[]' for nil:NilClass
```
Bang! Journey to the dependency hell starts.

The fix, from [RDoc is not a full Ruby parser and will fail when fed invalid ruby
programs. (parsing concern) #786][rdoc], is to upgrade gem rdoc
```
qy@Yis-MacBook-Air:~$ gem install --user-install rdoc
Fetching rdoc-6.3.3.gem
WARNING:  You don't have /Users/qy/.gem/ruby/2.6.0/bin in your PATH,
          gem executables will not run.
Successfully installed rdoc-6.3.3
Parsing documentation for rdoc-6.3.3
Installing ri documentation for rdoc-6.3.3
Done installing documentation for rdoc after 1 seconds
1 gem installed
```

This time, installation succeeds.
```
qy@Yis-MacBook-Air:~$ gem install --user-install bundler jekyll
WARNING:  You don't have /Users/qy/.gem/ruby/2.6.0/bin in your PATH,
          gem executables will not run.
Successfully installed bundler-2.3.1
Parsing documentation for bundler-2.3.1
Done installing documentation for bundler after 0 seconds
Successfully installed jekyll-4.2.1
Parsing documentation for jekyll-4.2.1
Installing ri documentation for jekyll-4.2.1
Done installing documentation for jekyll after 0 seconds
2 gems installed
```
I can see that some stuff are under my home directory, and I need to add one directory to the path search list
```
qy@Yis-MacBook-Air:~$ ls $HOME/.gem/ruby/2.6.0/bin/
bundle          jekyll          listen          ri              safe_yaml
bundler         kramdown        rdoc            rougify

qy@Yis-MacBook-Air:~/my_blog$ export PATH="$HOME/.gem/ruby/2.6.0/bin:$PATH"
```

# ffi issue
First run of jekyll fails, because of issue in ffi
```
qy@Yis-MacBook-Air:~/my_blog$ jekyll new --skip-bundle .
/Users/qy/.gem/ruby/2.6.0/gems/ffi-1.15.4/lib/ffi/library.rb:275: [BUG] Bus Error at 0x0000000100c68000
ruby 2.6.8p205 (2021-07-07 revision 67951) [universal.arm64e-darwin21]

-- Crash Report log information --------------------------------------------
   See Crash Report log file under the one of following:
     * ~/Library/Logs/DiagnosticReports
     * /Library/Logs/DiagnosticReports
   for more details.
Don't forget to include the above Crash Report log file in bug reports.

-- Control frame information -----------------------------------------------
c:0027 p:---- s:0156 e:000155 CFUNC  :attach
c:0026 p:0258 s:0150 e:000149 METHOD /Users/qy/.gem/ruby/2.6.0/gems/ffi-1.15.4/lib/ffi/library.rb:275
c:0025 p:0023 s:0130 e:000129 METHOD /Users/qy/.gem/ruby/2.6.0/gems/sassc-2.4.0/lib/sassc/native.rb:40
...

c:0002 p:0109 s:0008 E:001460 EVAL   /Users/qy/.gem/ruby/2.6.0/bin/jekyll:23 [FINISH]
c:0001 p:0000 s:0003 E:002680 (none) [FINISH]

-- Ruby level backtrace information ----------------------------------------
/Users/qy/.gem/ruby/2.6.0/bin/jekyll:23:in `<main>'
/Users/qy/.gem/ruby/2.6.0/bin/jekyll:23:in `load'
...
/Users/qy/.gem/ruby/2.6.0/gems/ffi-1.15.4/lib/ffi/library.rb:275:in `attach'

-- Other runtime information -----------------------------------------------

* Loaded script: /Users/qy/.gem/ruby/2.6.0/bin/jekyll

* Loaded features:

    0 enumerator.so
    1 thread.rb
    2 rational.so
    3 complex.so
    4 /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/universal-darwin21/enc/encdb.bundle
...
  412 /Users/qy/.gem/ruby/2.6.0/gems/sassc-2.4.0/lib/sassc/native/string_list.rb

[NOTE]
You may have encountered a bug in the Ruby interpreter or extension libraries.
Bug reports are welcome.
For details: https://www.ruby-lang.org/bugreport.html

[IMPORTANT]
Don't forget to include the Crash Report log file under
DiagnosticReports directory in bug reports.

Abort trap: 6
```

check the GEM environment before my trek in search results, seems like OK:
```
qy@Yis-MacBook-Air:~/my_blog$ gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 3.0.3.1
  - RUBY VERSION: 2.6.8 (2021-07-07 patchlevel 205) [universal.arm64e-darwin21]
  - INSTALLATION DIRECTORY: /Library/Ruby/Gems/2.6.0
  - USER INSTALLATION DIRECTORY: /Users/qy/.gem/ruby/2.6.0
  - RUBY EXECUTABLE: /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/bin/ruby
  - GIT EXECUTABLE: /opt/local/bin/git
  - EXECUTABLE DIRECTORY: /usr/local/bin
  - SPEC CACHE DIRECTORY: /Users/qy/.gem/specs
  - SYSTEM CONFIGURATION DIRECTORY: /Library/Ruby/Site
  - RUBYGEMS PLATFORMS:
    - ruby
    - universal-darwin-21
  - GEM PATHS:
     - /Library/Ruby/Gems/2.6.0
     - /Users/qy/.gem/ruby/2.6.0
     - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :backtrace => false
     - :bulk_threshold => 1000
  - REMOTE SOURCES:
     - https://rubygems.org/
  - SHELL PATH:
     - /Users/qy/.gem/ruby/2.6.0/bin
     - /opt/local/bin
     - /opt/local/sbin
     - /Users/qy/my_repo/bin/mac/my_mac
     - /Users/qy/my_root/usr/local/bin
     - /Users/qy/bin/mac
     - /opt/local/bin
     - /opt/local/sbin
     - /Users/qy/my_repo/bin
     - /usr/local/bin
     - /usr/bin
     - /bin
     - /usr/sbin
     - /sbin
     - /Library/TeX/texbin
     - /Library/Apple/usr/bin
     - /opt/local/bin
     - /opt/local/sbin
     - /Users/qy/my_repo/bin/mac/my_mac
     - /Users/qy/my_root/usr/local/bin
     - /Users/qy/bin/mac
     - /Users/qy/my_repo/bin
     - /Users/qy/bin
     - /Users/qy/bin/EMC/DD
     - /Users/qy/configuration/vim/bin
     -
     - /Users/qy/bin
     - /Users/qy/bin/EMC/DD
     - /Users/qy/configuration/vim/bin
```

Upgrade ffi like RDoc doesn't work
```
qy@Yis-MacBook-Air:~$ gem install --user-install ffi
WARNING:  You don't have /Users/qy/.gem/ruby/2.6.0/bin in your PATH,
          gem executables will not run.
Building native extensions. This could take a while...
Successfully installed ffi-1.15.4
Parsing documentation for ffi-1.15.4
Done installing documentation for ffi after 0 seconds
1 gem installed
```

In [Segmentation fault on M1 macOS Big Sur #864][ffi-solution], xhacker has a great and simple solution:
```
qy@Yis-MacBook-Air:~$ gem install --user-install ffi -- --enable-libffi-alloc
WARNING:  You don't have /Users/qy/.gem/ruby/2.6.0/bin in your PATH,
          gem executables will not run.
Building native extensions with: '--enable-libffi-alloc'
This could take a while...
Successfully installed ffi-1.15.4
Parsing documentation for ffi-1.15.4
Done installing documentation for ffi after 0 seconds
1 gem installed
```

Now, it works, but I need to run it with "--force" because there has already been some files under this directory
```
qy@Yis-MacBook-Air:~/my_blog$ jekyll new --skip-bundle .
          Conflict: /Users/qy/my_blog exists and is not empty.
                    Ensure /Users/qy/my_blog is empty or else try again with `--force` to proceed and overwrite any files.

qy@Yis-MacBook-Air:~/my_blog$ jekyll new --help
jekyll new -- Creates a new Jekyll site scaffold in PATH

Usage:

  jekyll new PATH

Options:
            --force        Force creation even if PATH already exists
            --blank        Creates scaffolding but with empty files
            --skip-bundle  Skip 'bundle install'
        -s, --source [DIR]  Source directory (defaults to ./)
        -d, --destination [DIR]  Destination directory (defaults to ./_site)
            --safe         Safe mode (defaults to false)
        -p, --plugins PLUGINS_DIR1[,PLUGINS_DIR2[,...]]  Plugins directory (defaults to ./_plugins)
            --layouts DIR  Layouts directory (defaults to ./_layouts)
            --profile      Generate a Liquid rendering profile
        -h, --help         Show this message
        -v, --version      Print the name and version
        -t, --trace        Show the full backtrace when an error occurs
qy@Yis-MacBook-Air:~/my_blog$ ls
Markdown_example_Marked2.md     README.md                       _config.yml
qy@Yis-MacBook-Air:~/my_blog$ jekyll new --skip-bundle --force .
New jekyll site installed in /Users/qy/my_blog.
Bundle install skipped.
```

# Reference

[Jekyll on macOS][jekyll-mac]: about how to install Jekyll on macOS 

[Troubleshooting installation of Jekyll][Jekyll-tr]: points to [issue
encountered by me][ffi-issue]

[RDoc is not a full Ruby parser and will fail when fed invalid ruby
programs. (parsing concern) #786][rdoc]: gives the solution for RDoc issue. 

[Segmentation fault on M1 macOS Big Sur #864][ffi-solution]: xhacker gave the solution
using "--enable-libffi-alloc" in installation of ffi, it works for me.

[Can't install Jekyll on Mac OS Big Sur (Apple Silicon M1) #8576][ffi-another]: osvik
found out one solution which uses Macports + rbenv + ruby-build to install a
higher version of Ruby and gems. This guy also created one Docker image for this
purpose. 

[Using Jekyll with Bundler | Jekyll • Simple, blog-aware, static
sites][jekyll-with-bundler]: I think this way is better than the one used by me,
because it fully leverages the power of Bundler: Jekyll is only installed to the
project directory. 

[Creating a GitHub Pages site with Jekyll][gh-pages-Jekyll]


[jekyll-mac]: https://jekyllrb.com/docs/installation/macos/
[jekyll-tr]: https://jekyllrb.com/docs/troubleshooting/
[ffi-issue]: https://github.com/ffi/ffi/issues/870 
[rdoc]: https://github.com/ruby/rdoc/issues/786
[ffi-solution]: https://github.com/ffi/ffi/issues/864
[ffi-another]: https://github.com/jekyll/jekyll/issues/8576
[jekyll-with-bundler]: https://jekyllrb.com/tutorials/using-jekyll-with-bundler/
[gh-pages-jekyll]: https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll
