title: Fast Copying Timestamp on Mac
date: 2014-08-18 18:56:19
tags: Mac shell
---
When I'm using Googe Docs to write logs for my lab work, it'll be much more convenient to insert timestamp and I'm surprised that they don't have this feature (Microsoft Word has it). 

But if you are using Mac OSX, this would be very easy to achive. Since there's a pretty good command line tool called `date` that can print timestamp in multiple formats. The thing we need to do is to copy the standard output to the clipboard. 

There are 2 ways to do this:

* Using `Alfred`. `Alfred` is a tool to quick launch applications and it has a feature of quick run customized shell scripts. However, this feature is together with its `Power Pack`, which is limited to paid users. 
* Using `automator`. `Automator` is a tool provided by Mac OSX that can pack scripts or recorded actions into an application or service. Here we choose this method.

Step:

1. launch `Automator` in `Launcher`. 
2. In menu bar, choose `File` -> `New`, then choose `Application`.
3. In the search bar search for `Run Shell Script`.
4. In the right window, put in the following script:

        date |tr -d '\n'| pbcopy

5. In menu bar, choose `File` -> `Save` to save your application, better to save it to `/Applications` for `Spotlight` to index. Here we name it as `TimeStamp`.

Voila! Now when you are using Google Docs or any other text editor, use `Spotlight` or `Alfred` to launch the application we just created, and press key `Command` + `V` to paste the timestamp to the document.

Explaining the script:

First `date` command will execute the `date` program and its output will be redirected to `tr` program, which with argument `-d '\n'`, will delete the last "endline" character. At last, the result will be past to `pbcopy` program to add it to the clipboard.

Further more, we can also add arguments to `date` to customize the format of the timestamp. Here is a nice article about this. [Link.](http://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/) For myself, I use this `date +"%d/%m/%y %a %H:%M"`, which print something like this: `18/08/14 Mon 20:25` .