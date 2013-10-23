---
layout: post
title: INTEGRATING GRUNT AND LESS
description: "Integrating Grunt tasks runner with Less compiler"
modified: 2013-09-06
tags: [grunt, less]

comments: true
share: true
---

This is the first post I write for our new brand blog and I would like to start talking about integrating Grunt and Less. I know a lot of you (including me) love Sass and use it over Less and perhaps you read the CSS-Tricks article about Sass vs LESS (if you didn’t I hardly advise you to do it). But there are also a lot of Less fans out there and I would like to show the options to set up Grunt with Less.

Assuming you already have nodejs, npm and grunt running in your project, if you don’t you can check the getting started section in grunt site [http://gruntjs.com/getting-started](http://gruntjs.com/getting-started) or the Justin MacCandless tutorial [http://is.gd/jrtajr](http://is.gd/jrtajr)


USING GRUNT LESS TASK
--------------------

This is the simpler way to set up less into Grunt, all we need to know is load the task into Gruntfile and set some parameters.

First we need to install grunt-contrib-less in your grunt project directory

{% highlight javascript %}
npm install grunt-contrib-less --save-dev
{% endhighlight %}

Then we need to go to our Gruntfile.js with our favorite text editor and add the task to the end of the file

{% highlight javascript %}
grunt.loadNpmTasks('grunt-contrib-less');
{% endhighlight %}

Then we need to add the "less" task to grunt initconfig 

{% highlight javascript %}
less: {
    development: {
        options: {
            paths: ["assets/css"]
        },
        files: {
            "path/to/result.css": "path/to/source.less"
        }
    }
}
{% endhighlight %}

Note we define the development environment and then we define files to be compiled into a result.css file, the second files parameter can be an array of files if you want to compile different files into one.

You can run the task with

{% highlight javascript %}
grunt less
{% endhighlight %}

USING GRUNT RECESS
------------------

Recess is a Twitter code quality tool which combines lint, minifying and compilation css and less files for projects. It uses the lesscss compiler and other tools to minify and lint CSS.

First you need to install the grunt task with npm

{% highlight javascript %}
npm install grunt-recess --save-dev
{% endhighlight %}


then you need to add the task to your Gruntfile.js

{% highlight javascript %}
grunt.loadNpmTasks('grunt-recess');
{% endhighlight %}

and add the task to grunt initconfig

{% highlight javascript %}
recess: {
    dist: {
        options: {
            compile: true
        },
        files: {
            path/to/result.css": ["path/to/source.less"]
        }
    }
}
{% endhighlight %}

Once again we can use the second parameter as an array to compile several files into a single one.

You can run the task with

{% highlight javascript %}
grunt recess
{% endhighlight %}

USING WATCH TO AUTOCOMPILE LESS
--------------------------

**NOTE:** There was a known bug with watch where watch stopped running tasks after first or second file changing which is already fixed in new versions, so you need to make sure you use the last watch version to avoid this issue.

Running the less/recess task every time you modify a less file is not the best way to work, we can use watch to keep an eye in our less files and call recess/less every time one of them changes.

First we need to install grunt-contrib-watch (if you don’t have it already)

{% highlight javascript %}
npm install grunt-contrib-watch
{% endhighlight %}


Then add watch to our gruntfile.js

{% highlight javascript %}
grunt.loadNpmTasks('grunt-contrib-watch');
{% endhighlight %}

And define the watch task (if defined before add the "less" subtask)

{% highlight javascript %}
watch: {
    less: {
        files: ['path/to/source/{,*/}*.less'],
        tasks: ['recess']
    }
}
{% endhighlight %}

You can change "recess" with "less" in case you use the contrib-less task. Now you just need to run

{% highlight javascript %}
grunt watch
{% endhighlight %}

and keep your terminal open and watch will run the less compiler task every time you change a file.