title: Shell Script for Hexo source folder backup
date: 2014-04-17 20:45:20
tags: [hexo]
---

[Hexo](http://hexo.io) is a very good tool to generate blog pages from mardown files, and it is more comvinient together with Github. But everytime I deploy a new post to Github using `hexo deploy`, it only commits generated .html files to the repo. 

Those original `.md` articles however seems very important to me for revising contents in the future, and it would be troublesome if those `.md` files were somehow lost. 

In order to keep them save and track my every changes, I decide to find a way to put them altogether in the repo with the deployed .html files. So I wrote a shell script the achieve this.

In blog project folder:
{% codeblock lang:shell %}
$ vim push.sh
{% endcodeblock %}
add content:

{% code lang:shell %}
#!/bin/bash

hexo generate
if test -d public/source_backup
then
    rm -rf public/source_backup
fi
cp -r source/ public/source_backup
hexo deploy
{% endcode %}

add executable:

{% codeblock lang:shell %}
$ chmod +x push.sh
{% endcodeblock %}

This script first generate "ready to publish" page files, and then check if previous version of source_backup directory exit. If exits, delete it and copy the latest version, then deploy it. `hexo deploy` command will automatically add everything in public/ folder and commit the changes.

Next time use this command to deploy:

{% codeblock lang:shell %}
$ ./push.sh
{% endcodeblock %}

Therefore, we have our original `.md` files safely tracked in GitHub, we can easily make changes to the posts on whatever version in the past you want. You can have the script copied to other directory if you have other Hexo projects, or add them to your `.bash_profile` script.
