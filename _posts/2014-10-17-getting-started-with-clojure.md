---
layout:     post
title:      Getting Started With Clojure
date:       2014-10-17 14:00:00
categories:
  - clojure

---

I recently found some [interesting work](https://github.com/garethr/digitalocean-expect) going on in terms of using Clojure code to represent policy, or perhaps intent, of infrastructure (ie. tiered web apps and the like), and I wanted to try it out. That meant getting a Clojure environment up and running, so that is what is in this post: getting a basic Clojure environment on OSX, compiling a jar, and running the Clojure shell (REPL).

<!-- more -->

There is a big list of books, articles, tutorials and other documentation on the [clojure website](http://clojure.org/getting_started).

Also--[Clojure Koans](https://github.com/functional-koans/clojure-koans) might be an even easier way to get started with Clojure.

### Java

Most of the time I just refuse to install Java. However, if I want to checkout Clojure there's not much choice. So here we go.

I installed [Java for OSX](http://support.apple.com/kb/dl1572). I'm just using the Java that Apple provides. Please note that I didn't do any research on the best Java SDK to use or anything, essentially just used the first link that came up, which was Apple's package.

```bash
curtis$ java -version
java version "1.6.0_65"
Java(TM) SE Runtime Environment (build 1.6.0_65-b14-466.1-11M4716)
Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-466.1, mixed mode)
```

### Install and use lein

At this point I'm mostly following the [getting started](http://www.braveclojure.com/getting-started/) section of ["Clojure for the Brave and True"](http://www.braveclojure.com/) and some from ["Clojure from the ground up"](http://aphyr.com/posts/301-clojure-from-the-ground-up-welcome), a series of posts by a [very smart man](https://twitter.com/aphyr).

After getting a Java SDK, I installed ```lein```.

```bash
curtis$ pwd
/usr/local/bin
curtis$ curl https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein > lein
curtis$ sudo chmod a+x /usr/local/bin/lein
```

The first time running lein is going to download a few requirements.

```bash
curtis$ lein version
Downloading Leiningen to /Users/curtis/.lein/self-installs/leiningen-2.5.0-standalone.jar now...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   406    0   406    0     0    746      0 --:--:-- --:--:-- --:--:--   746
100 14.2M  100 14.2M    0     0  1128k      0  0:00:12  0:00:12 --:--:-- 1425k
Leiningen 2.5.0 on Java 1.6.0_65 Java HotSpot(TM) 64-Bit Server VM
curtis$ lein version
Leiningen 2.5.0 on Java 1.6.0_65 Java HotSpot(TM) 64-Bit Server VM
```

After that first run we can now use lein to create an application skeleton.

```bash
curtis$ lein new app clojure-noob
Generating a project called clojure-noob based on the 'app' template.
curtis$ ls
clojure-noob
curtis$ cd clojure-noob/
curtis$ tree
.
├── LICENSE
├── README.md
├── doc
│   └── intro.md
├── project.clj
├── resources
├── src
│   └── clojure_noob
│       └── core.clj
└── test
    └── clojure_noob
        └── core_test.clj

6 directories, 6 files
```

I changed ```src/clojure_noob/core.clj``` to be:

```bash
(ns clojure-noob.core
  (:gen-class))

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println "I'm a little tea pot"))
```

### Run some code

Next I ran the small Clojure program with ```lein run```.

```bash
curtis$ pwd
/Users/curtis/working/clojure/clojure-noob
curtis$ lein run
Retrieving org/clojure/clojure/1.6.0/clojure-1.6.0.pom from central
Retrieving org/sonatype/oss/oss-parent/7/oss-parent-7.pom from central
Retrieving org/clojure/tools.nrepl/0.2.6/tools.nrepl-0.2.6.pom from central
Retrieving org/clojure/pom.contrib/0.1.2/pom.contrib-0.1.2.pom from central
Retrieving clojure-complete/clojure-complete/0.2.3/clojure-complete-0.2.3.pom from clojars
Retrieving org/clojure/tools.nrepl/0.2.6/tools.nrepl-0.2.6.jar from central
Retrieving org/clojure/clojure/1.6.0/clojure-1.6.0.jar from central
Retrieving clojure-complete/clojure-complete/0.2.3/clojure-complete-0.2.3.jar from clojars
I'm a little tea pot
curtis$ lein run #second run
I'm a little tea pot
curtis$
```

Next we can compile it into a jar.

```bash
curtis$ lein uberjar
Compiling clojure-noob.core
Created /Users/curtis/working/clojure/clojure-noob/target/uberjar/clojure-noob-0.1.0-SNAPSHOT.jar
Created /Users/curtis/working/clojure/clojure-noob/target/uberjar/clojure-noob-0.1.0-SNAPSHOT-standalone.jar
```

And that jar can be run:

```bash
curtis$ time java -jar target/uberjar/clojure-noob-0.1.0-SNAPSHOT-standalone.jar
I'm a little tea pot

real	0m0.893s
user	0m1.264s
sys	0m0.076s
curtis$ lein run
I'm a little tea pot
curtis$ time lein run
I'm a little tea pot

real	0m2.288s
user	0m2.421s
sys	0m0.246s
```

### REPL

Clojure also has a kind of shell: a "read-eval-print-loop" ([REPL](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)). I love languages that have a shell of sorts, like python (and even better, [ipython](http://ipython.org/)) irb in ruby, and others. It's great to be able to quickly try things out and explore in a language shell.

```bash
curtis$ lein repl
nREPL server started on port 52089 on host 127.0.0.1 - nrepl://127.0.0.1:52089
REPL-y 0.3.5, nREPL 0.2.6
Clojure 1.6.0
Java HotSpot(TM) 64-Bit Server VM 1.6.0_65-b14-466.1-11M4716
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
    Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

clojure-noob.core=>
```

The most simple command we can run is...

```bash
clojure-noob.core=> nil
nil
```

Just for fun, there is a verb, ```inc```, that will increment things.

```bash
clojure-noob.core=> inc
#<core$inc clojure.core$inc@73ae9565>
clojure-noob.core=> 'inc
inc
clojure-noob.core=> (inc 1)
2
clojure-noob.core=> (inc (inc 2))
4
```

### Conclusion

At this point we have a basic Clojure environment up and running, one that can be used to explore all the [cool Clojure code out there](https://github.com/trending?l=clojure). I'm not a big fan of Java, but the JVM obviously allows some interesting applications.
