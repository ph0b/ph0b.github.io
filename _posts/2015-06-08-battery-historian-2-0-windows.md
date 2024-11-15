---
id: 203
title: 'Getting Battery Historian 2.0 to work on Windows'
date: '2015-06-08T17:35:41+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=203'
permalink: /battery-historian-2-0-windows/
nkweb_code_in_head:
    - default
nkweb_Use_Custom_js:
    - default
nkweb_Custom_js:
    - ''
nkweb_Use_Custom_Values:
    - default
nkweb_Custom_Values:
    - ''
nkweb_Use_Custom:
    - 'false'
nkweb_Custom_Code:
    - ''
categories:
    - Android
tags:
    - Android
    - 'Battery Historian'
    - Windows
---

**Last updated on 3/25/16**

If you’ve never heard about Battery Historian, you can first have a look at the [official page](https://github.com/google/battery-historian) and some of [Android Performance Patterns videos](https://www.youtube.com/watch?v=4D7_N2XEw20). Then, come back here if you’re using Windows, to get ready fixing your app 🙂

The first version of Battery Historian released last year was a simple cross-platform Python script. Two weeks ago, during Google I/O 15, a new version of Battery Historian (2.0!) has been quietly released.

This new release brings a lot of welcome improvements: more clarity in the reports, per-app summaries, a quicker execution, the presence of a Readme, etc.

While all these new features are awesome, Battery Historian got more complicated to install because of the addition of many dependencies and all its documentation and internal scripts are written for Unix compatible systems (Linux and Mac OSX). So out of the box, it’s simply not working on Windows.

Last week, I held a talk at Droidcon Berlin on power optimization for Android, and I’ve made a quick poll to the audience: 30-40% were actually developing on Windows (me included, 50%+ of the time).

So if you’re an Android developer who is developing on Windows, let me make you gain some time, here is my short guide on how to install Battery Historian 2.0 on Windows:

## 1. Installing the dependencies

### Git

While you may already have some version of git installed on your system, it will be easier if you get the official client and make it available to Windows Command Prompt. In order to do so, download [git’s latest version](http://git-scm.com/download/win) and trigger its installation.

When you get the chance to do it, do not forget to pick “Use Git from the Windows Command Prompt” :  
[![select Use Git from the Windows Command Prompt](/wp-content/uploads/2015/06/battery-historian-1.png)](/wp-content/uploads/2015/06/battery-historian-1.png)

### Python

Battery Historian **1.0** requires Python **2.7**, and Battery Historian **2.0** works with both Python **2.7** and **3.0** (with a small fix).  
If you want to be able to generate and look at 1.0 reports from Battery Historian 2.0 (the interface gives you this possibility – but it’s not that interesting in my view), use Python 2.7, else you’re free to choose Python 3.0.  
Download and trigger the installer of your choice from [Python.org](https://www.python.org/downloads/windows/).

### Go

Battery Historian 2.0 has been rewritten in Go, so you need first to [download and install](http://golang.org/doc/install) it.  
Once this is done, create a go workspace (ie. a directory):

```shell
mkdir %USERPROFILE%\go-workspace
```

## 2. Installing Battery Historian 2.0

Open a Windows Command Prompt (cmd) and set useful Go specific environment variables, then go into your Go workspace:

```shell
set GOPATH=%USERPROFILE%\go-workspace
set GOBIN=%GOPATH%\bin
set PATH=%PATH%;%GOBIN%
cd %GOPATH%
```

Get the latest battery historian and install it:

```shell
go get -u github.com/google/battery-historian
cd %GOPATH%\src\github.com\google\battery-historian
go run setup.go
```

***update from 3/25/16**: Battery historian has been updated with a much improved support for Windows, so the following method isn’t required anymore:*

~~Now get additional go dependencies (proto, protoc-gen-go) as well as Battery Historian, by executing:~~

```shell
go get -u github.com/golang/protobuf/proto
go get -u github.com/golang/protobuf/protoc-gen-go
go get -u github.com/google/battery-historian
```

~~Then you need to add closure library and compilers to battery historian. Go into battery-historian: *%GOPATH%\\src\\github.com\\google\\battery-historian* and create two directories: *third\_party* and *compiled*.~~

~~Download the [Closure compiler](http://dl.google.com/closure-compiler/compiler-20150315.zip) and unzip it inside *third\_party\\closure-compiler*  
 ![Closure Compiler unziped](/wp-content/uploads/2015/06/Capture.png)~~

~~Clone the closure library to *third\_party\\closure-library*~~

```shell
git clone https://github.com/google/closure-library third_party/closure-library
```

~~If you’re using Python 3, change the ligne 123 of *third\_party/closure-library\\closure\\bin\\build\\source.py* from this library: (that fix has been upstreamed)~~

```diff
-    fileobj = open(path)
+    fileobj = open(path, encoding="utf8")
```

~~While we’re editing sources, if you want the Battery Historian 1.0 tab to work from Battery Historian 2.0, change line 266 of *battery-historian\\analyzer\\analyzer.go* to add first the reference to your Python executable (python.exe is enough if the directory is part of your path):~~

```shell
-       cmd := exec.Command("./historian.py", "-c", "-m", "-r", reportName, filepath)
+       cmd := exec.Command("C:\\Python27\\python.exe", "./historian.py", "-c", "-m", "-r", reportName, filepath)
```

~~Finally, to finish the installation of Battery Historian 2.0, execute these two commands from the *battery-historian* directory:~~

```shell
third_party\closure-library\closure\bin\build\depswriter.py --root="third_party\closure-library\closure\goog" --root_with_prefix="js ../../../../js" > compiled\historian_deps-runfiles.js

java -jar third_party\closure-compiler\compiler.jar --closure_entry_point historian.Historian --js js\*.js --js third_party\closure-library\closure\goog\base.js --js third_party\closure-library\closure\goog\** --only_closure_dependencies --generate_exports --js_output_file compiled\historian-optimized.js --compilation_level SIMPLE_OPTIMIZATIONS
```

## 3. Running Battery Historian 2.0

Now that you’ve went through all the installation steps, you can finally launch Battery Historian:

```shell
set GOPATH=%USERPROFILE%\go-workspace
cd %GOPATH%\src\github.com\google\battery-historian
go run cmd\battery-historian\battery-historian.go
```

*Tip*: place these lines inside a *.cmd* file you’ll be able to double click on to launch Battery Historian.

[![battery-historian.cmd](/wp-content/uploads/2015/06/Capture1-1024x367.png)](/wp-content/uploads/2015/06/Capture1.png)

Go to <http://localhost:9999> and you should see your Battery Historian 2.0 local server up and running:

[![Battery Historian Server](/wp-content/uploads/2015/06/Capture2-300x156.png)](/wp-content/uploads/2015/06/Capture2.png)

Now you can upload a bugreport to it, and enjoy this tool on Windows!

[![Battery Historian 2.0 report](/wp-content/uploads/2015/06/Screenshot-5-1024x795.png)](/wp-content/uploads/2015/06/Screenshot-5.png)
