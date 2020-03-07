---
layout: post
title: "Making a Command Line Tool Npm Package"
date: 2015-08-19
---

This article is meant to be a quick tutorial on creating a globally installed
npm package that provides a simple command line tool to users. Our command line
tool will be a clone of the `ls` command line tool, which prints a list of the
contents of a directory.

## Setting Up the Package

Start off by creating a directory to house your package and then moving into
that directory. Name it whatever you want.

```
mkdir list-tool
cd list-tool
```

Then you'll want to create a package.json file, which you can easily do by
running:

```
npm init
```

You'll be asked a series of questions, and you can use the default values for
all of them, or specify alternate values if you wish. When you are ready to
publish your package, you'll want to revisit your package.json to make sure all
the values are updated and correct.

For this package, we don't need to install any dependencies. However, if you do
need to install dependencies, you can run the following command to install and
save them in your package.json:

```
npm install --save <package name>
```

We only need to create 2 more files for this package, but both files need to be
housed inside a `bin` directory inside of our package's directory. So run the
following commands from inside the root directory of the package:

```
mkdir bin cd bin
touch list-tool
```

Your package directory should look something like this:

```
list-tool
│ list-tool
│ package.json
```

Before we get to the code, there are 2 properties you'll need to add to your
package.json: `bin` and `preferGlobal`.

```
"bin": {
  "list-tool": "list-tool"
},
"preferGlobal": "true"
```

`bin` tells npm where your executable file is so that it can install it into
the PATH. And `preferGlobal`, if set to `true`, will warn users if they try to
install the package locally. `preferGlobal` is not necessary, but it can be
nice to let users know that the package is meant to be installed globally.

## Creating the Package

Now we have the basics set up for our package. Let's get into the code.

Open up the list-tool file. This is our executable file, so we first need to tell the machine how to run this file:

```
!/usr/bin/env node
```

Now let's require in the built-in Node.js `fs` module.

```
var fs = require('fs');
```

Then, we are going to need to read the filenames in the current directory. To
do this, we are going to use the the synchronous version of the `fs.readdir`
method, which is `fs.readdirSync`. The reason we will use the synchronous
version is because this is a simple program, and all we need to do is read the
filenames and print them out. No need for asynchronous operations.

The code for this should look like:

```
var files = fs.readdirSync(process.cwd());
```

If you didn't guess already, `process.cwd()` returns the current working
directory. This is important because our command line tool should work
correctly no matter where the tool is being called.

`fs.readdirSync` will return an array of all the filenames. So all we need to
do now is print them out one-by-one. You can do this using the
`Array.prototype.forEach` method:

```
files.forEach(function(file) {
  console.log(file);
});
```

It's that easy. So all of the code together is:

```
var fs = require('fs');
var files = fs.readdirSync('.');files.forEach(function(file) {
  console.log(file);
});
```
