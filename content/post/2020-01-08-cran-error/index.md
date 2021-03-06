---
slug: cran-error
title: "From Shock to Competence: How Not to Panic When You Receive E-mail from CRAN about Failed Checks"
authors:
  - Julia Romanowska
date: "2020-01-08"
tags:
- package development
output: 
  html_document:
    keep_md: true
---

> This post was contributed by [Julia Romanowska](https://jrom.bitbucket.io/homepage/), Researcher at the University of Bergen, Norway. Thank you, Julia!

> Edited on 2020-01-09




I'm involved in development of the [Haplin](https://folk.uib.no/gjessing/genetics/software/haplin) R package, which enables fast genetic association analyses (very useful for those involved in genetic epidemiology research). We are a team of scientists that have various background, from genetics, through bioinformatics and statistics. Here's a short text about our latest "adventure" with CRAN and how it taught us some useful stuff.

## OMG, CRAN wrote to me!

There will come the time when you check your mailbox and see this dreaded message about a check that your beloved CRAN package failed. Even though the deadline for submitting a fix is quite reasonable, you might imagine yourself sitting long into the night over your laptop just to find out where is this one comma that was the source of the error. However, I need to say we were positively surprised over the level of details of feedback we got from CRAN concerning the error in _our_ package.

In our case, the error was related to the changes that will be introduced in the next R version and that include throwing an error when the ``if`` statement resolves to a vector of values instead of one boolean value. In our code, we used to test for specific type of input data using ``class( obj ) == "myClass"``. This is sometimes resolved to a vector, which thus leads to an error in the `r-devel` CRAN tests. [This blog post](https://developer.r-project.org/Blog/public/2019/11/09/when-you-think-class.-think-again/index.html) explains it in more details.

Thus, important :sparkles: tip \#1 :sparkles: *read the e-mail from CRAN carefully!*

## Reproducing the error

OK, let's get to work! Firstly, check the "checks" webpage on CRAN. These CRAN checks come in different _flavors_ and hopefully, not all checks failed. In our case, the flavor that produced the error was [`r-devel-linux-x86_64-debian-clang`](https://blog.r-hub.io/2019/04/25/r-devel-linux-x86-64-debian-clang/) and I had no wish to install the developmental version of R and other packages on my local laptop just to try mimicking this setup. That's when I thought about checking [*Docker*](https://www.docker.com/).

Installing `Docker` and running a test went well, but what one needs is a VM that would be exactly the same as the platforms CRAN uses for testing. I realized that it would take too much time to re-create it, so again my DuckDuckGo search engine[^1] came to the rescue.

:sparkles: Tip \#2 :sparkles: *search the net!*

## R and Docker

<!-- r-hub and rocker - which is good for what -->
There are three services that are worth mentioning:

- [winbuilder](https://win-builder.r-project.org/), which is running on the same Windows machine with the same setup as the CRAN incoming submission checks, works from any operating system, and is very useful for reproducing CRAN errors.  It's also _recommended_ to be used prior to CRAN submissions, (Edited on 2020-01-09, thanks to a comment by [Henrik Bengtsson](https://github.com/HenrikBengtsson)).
- [rhub](https://r-hub.github.io/rhub/), which is an R package for using the [R-hub package builder](https://docs.r-hub.io/#package-builder),
- and the [Rocker project](https://www.rocker-project.org/), which distributes various Docker images useful for R and RStudio testing.

If you are a maintainer of an R package and don't want to install Docker locally, you will find the `rhub` package very useful, as you can push your package to the server for testing with one command! Check out e.g., [the package docs](https://r-hub.github.io/rhub/).

If you want to test more locally, and importantly, if you want to test visualizations, GUI, or specific behavior of the code in RStudio, you should take a look at the [Rocker project](https://www.rocker-project.org/). They provide easy-to-run Docker images with pre-installed R and RStudio, which you can play with through a browser window.

## R-hub locally

In my case, I wanted to test locally and not on Windows, so I decided to use one of the `rhub` functions, [`local_check_linux()`](https://r-hub.github.io/rhub/reference/local_check_linux.html). This seemed like a very easy way to run locally tests on a platform that is exactly the same as CRAN uses! And it is... only not in my case :wink:

After getting the same error for the nth time, I decided to get help [at the source](https://github.com/r-hub/rhub/issues/322). I tried several things, including digging into the `rhub` code locally, but nothing worked. Long story short - my Docker installation by default gave no access to internet to the containers it launched.

:sparkles: Tip \#3 :sparkles: *ask for help other people!* (they don't bite, especially not via e-mails)

## Script for semi-manual checks

Another couple of days and I talked with my husband, who showed me a script he used in one of his projects. [The script](https://bitbucket.org/Grantlab/bio3d/src/master/ver_devel/util/run_dockercheck.sh) uses a local Docker installation, fetches one of [the images hosted by R-hub](https://github.com/r-hub/rhub-linux-builders#rhub-linux-builders), and runs tests locally. *Bingo!* As this was the second time I tried using Docker, I needed some attempts to adjust the script to my purposes, but it finally worked!


## Conclusion

Since I could now reproduce the exact error and I knew roughly which parts of the code caused it, I fixed the problems relatively fast. The main issue, as mentioned above, was with the identification of a class of the objects, which I've re-written now to ``is( obj, "myClass" )`` (for S4 objects) and ``inherits( obj, "myClass" )`` (for S3 objects) (Edited on 2020-01-09, thanks to a comment by [Henrik Bengtsson](https://github.com/HenrikBengtsson)).

Thanks to `rhub` and other tools available online, it's relatively easy to test a CRAN package - just remember not to panic and read the check reports thoroughly.


### _Post scriptum_

While writing this post, I've come upon another R package [dockr](https://cran.r-project.org/package=dockr), which helps creating a Docker container with the package one wants to test already available within the container. I haven't tried it yet, but [it seems very promising](http://smaakage85.netlify.com/2019/12/21/dockr-easy-containerization-for-r/)!


> Find Julia on [GitHub](https://github.com/jromanowska), [Bitbucket](https://bitbucket.org/jrom/profile/repositories), or [Facebook](https://www.facebook.com/julia.romanowska.733)


[^1]: I repeatedly refuse to use Google [to protect my privacy](https://spreadprivacy.com/how-to-remove-google/)
