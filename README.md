# Overview

**Goal.** The goal of this tutorial is to make sure that you are able to
create a container of your code repository in such a way that yourself in the
future, or external collaborators, are able to fully reproduce your work.
Most importantly, you will do this in such a way that anyone will be able to
reproduce the work without having to worry about differences in software
versions today, or 100 years from now.

**Knowledge requirements.** This example is done entirely using free,
open-source software. In order for this to work relatively seamlessly, we expect
that you have good experience with the R language, git system of version control,
and some basic knowledge in bash. If you do not have any of the above skills,
this may be a bit out there for you, but please feel free to still give it a go---
we would love to hear your feedback on how to improve the content of this tutorial.

**Major steps.** In this tutorial you will:

1) Pull an existing example git repository hosted on GitHub containing R code that
runs a linear model on some dataset, and produces a .pdf output bivariate plot;

2) Learn how to build a Docker container from this repository, and run it
locally;

3) Push this container to Docker Hub;

4) Use Binder to build and run the Docker container remotely, which will be
done via an interactive RStudio session on your web browser. There you will have access
to the full content of the code repository, as well as the correct OS and package
versions. This approach will allow any user to run and fully reproduce the output
generated by the original code in the repository.

But first things first.

## The reproducibility crisis

One of the biggest challenges scientists face today is making sure that their research
is fully reproducible. This reproducibility has three main pillars. It starts with a fully
transparent plan of the experimental design and hypothesis to be tested ([see more here](https://osf.io/)),
moving onto the actual collection of the data ([see more here](https://en.wikipedia.org/wiki/Reproducibility_Project)),
to finally making sure that all analyses and output are fully reproducible from
scratch using the collected data and computer code. Here we will address the latter
form of reproducibility based exclusively on open-source software and free-of-charge
on-line platforms.

## Organised project structure

We will assume that you did maintain a record of your original research intention,
and that the data is fully collected and, most importantly, **untouched**. Raw
data should always be kept as read-only. Literally any modification that is needed
to be applied to a dataset can be done via computer code, which helps you keep
a fully transparent record of how the data was modified from original version to
analysis version used to answer the research question.

Once files are in place, we need to make sure that we maintain everything organised.
We recommend following a simple project directory structure as exemplified by the
[NiceRCode blog](https://nicercode.github.io/blog/2013-04-05-projects/). So for the
sake of this particular tutorial, we will assume that you have also modularised your
code into unit functions ([see examples in the R language](https://nicercode.github.io/2014-02-13-UNSW/lessons/10-functions/)),
and have been maintaining a full record of how the code has been modified through
time using [version control with git](https://nicercode.github.io/2014-02-13-UNSW/lessons/70-version-control/).
Also, make sure you have your own free [GitHub account](https://help.github.com/en/github/getting-started-with-github/signing-up-for-a-new-github-account).

## But my open-source coding software keeps changing version!

This is where we start approaching the utility of Docker containers. Many of you may
have experienced a situation where the code and project history are fully transparent,
have been deposited in an on-line, free repository, but can no longer be reproduced
because the software version and associated packages that you originally used are
outdated. How can we solve this issue, such that our code will always yield
the exact same output, even in a million years from now?

## Enter, Docker container!

As with anything in computer programming, every skill and technique comes packed
with horrible lingo. When it comes to Docker containers you will probably see the
words image and container being used a lot, so let's go ahead and get those
definitions out of our way.

An *image* is a static (unchangeable) file that bundles code and all its
dependencies such that your code repository runs reliably on, say, both
your original MacBook and your colleague's Windows PC. It contains
the necessary system libraries, code, runtime and system tools for this magic
to happen. However, an image is just a snapshot which serves as a template to
build a *container*. In other words, a container is a running image, and cannot
exist without the image, whereas an image can exist without a container.

*Docker* is a containerisation software which allows you to create lightweight,
standalone, executable images from which you can create containers to run your
code repository and fully reproduce the output. This essentially allows
a scientist to isolate their code repository from its environment, solving the third
pillar of our [reproducibility crisis](#the-reproducibility-crisis) section above.

## I am hooked! How does it work?

First of all, you need to download and [install Docker](https://docs.docker.com/get-docker/)
on your machine. This is basically the software that contains all the tricks for you to build 
your own containers. Once you finished installing it, open the software which will
contain OS-specific examples on how to build Docker containers. For this tutorial, we will
provide a Unix-based example, so it should work on any MacOS or Linux terminal (sorry Windows
users, please raise an issue and we will try to expand this tutorial for you as well).

1. We will start by forking a public git repository from GitHub. Make sure you are logged
into your own GitHub account. Then open the `AIMS/docker-example` repository page on your 
web browser: https://github.com/AIMS/docker-example. At the top right hand corner, there
is a option to "Fork" the repository. That will essentially make a full copy of the 
`AIMS/docker-example` repository on your own account, allowing you to modify the content as
much as needed without interfering with the history of the original `AIMS/docker-example`.

2. Now clone the forked repository from GitHub to a local folder of your choice on your
machine (below referred to as `path_of_your_choice`). The forked repository will be named
`username/docker-example`, where `username` corresponds to your own GitHub account name.
Make sure to substitute the appropriate names in the example code below.

  ```{git clone, engine='bash', results='markdown', eval=FALSE}
  cd path_of_your_choice
  git clone git@github.com:username/docker-example.git
  ```

3. Now locally navigate to the cloned repository

  ```{cd docker, engine='bash', results='markdown', eval=FALSE}
  cd docker-example
  ```

4. Do not run anything on it just yet. Before we get on to the Docker building
part, we need to have a look at the file structure in this repository. The key
files here are `DESCRIPTION`, `Dockerfile`, and `.dockerignore`. The `DESCRIPTION`
contains a general description of what this code repository contains, info about
the authors, the license (see [here](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/licensing-a-repository)
why you should always include a license with your public repository). It also
details the packages dependencies needed to to make the code run (in this
example, just [ggplot2](https://ggplot2.tidyverse.org/)).
The `Dockerfile` contains a set of instructions that Docker uses to create your
container with the correct specifications. While you do not need to know all
the bits and pieces here, the two main aspects needed at this stage at the very top
and very bottom content of this file. The three first rows contain information
on what version of software you want (we use the [`rocker/verse:3.6.3` container](https://hub.docker.com/r/rocker/verse) freely provided
by the [rocker](https://github.com/rocker-org/rocker-versioned) team), as well as
information about yourself. The bottom info contains the exact date from which the
R version and associated packages should be downloaded. In this example, we use an
[MRAN](http://mran.revolutionanalytics.com/) image which contains version 3.6.3
and all package versions on the 31/03/2020. The `.dockerignore` plays essentially
the same role as the `.gitignore` file  on your version control system. It tells
Docker which files folder should be ignored when building the container.

5. Now that we're happy about the basic set up, make sure that Docker is open
and running on your local machine.

6. We're ready to build! Make sure you're on the `docker-example` path, and run
(this may take several minutes to set up):

  ```{docker build, engine='bash', results='markdown', eval=FALSE}
  docker build -t docker-example .
  ```

7. Once the container is ready, you can launch it using the following code
(it will map your current working directory inside the docker container):

  ```{docker run, engine='bash', results='markdown', eval=FALSE}
  docker run --user root -v $(pwd):/home/rstudio/ -p 8787:8787 -e DISABLE_AUTH=true docker-example .
  ```

This line initialises the docker container, and you can physically inspect it and run it on an
RStudio session via your web browser which points to [localhost:8787](http://localhost:8787).

**NB:** Any changes you make to the code via the browser will automatically change
the code on the original folder on your machine. If this is not wanted behaviour, you may want
to first re-clone your GitHub repo on a new folder in your machine, and then build the container
image (i.e. step 6 above) from within this new re-cloned local repo to play around via the web
browser (i.e. step 7 above).

This repository is properly structured following the [NiceRCode guidelines](#organised-project-structure).
You should be able to simply `source("analysis.R")` in the RStudio console, which will generate a
new output on the newly-created `output` folder.

## This is awesome! How can I share my container with colleagues?

This is essentially the last part of our tutorial. You may be happy with building
your Docker container locally, but you may also want to make it accessible to your
colleagues who are less versed in these tools. You have three options, in increasing
level (not that much) of time investment:

A) One option (harder for colleagues, easier for you) would be for them to also
follow steps 2--7 above (they don't need to fork your GitHub repo as
long as they don't try to push back to it -- they won't have the permissions to do so
unless you give it to them).

B) Another option is to push the container to [Docker Hub](https://hub.docker.com/),
similar to how one would push code to their repository on GitHub. To do so, first make
sure you have created an account with Docker Hub, and that they are logged into this
account locally on their Docker app. To make things easy to remember and consistent,
I would try to have the account name be the same as your GitHub account name. Then,
simply go back to the Terminal and type:

  ```{docker push, engine='bash', results='markdown', eval=FALSE}
  docker tag docker-example username/docker-example
  docker push username/docker-example
  ```

remember to replace `username` with your actual Docker Hub account name. You can
check that the container is now hosted on your [Docker Hub repositories](https://hub.docker.com/repositories).
Your colleagues can then pull the Docker container locally on their machines, and can
simply run it (i.e. step 7 in the [above section](#i-am-hooked-how-does-it-work)). That requires them to
also have a GitHub account and the Docker app installed on their machines. They have
to clone the GitHub repo first, then pull the Docker container you created,
navigate to the correct folder, then run it:

  ```{docker clonerun, engine='bash', results='markdown', eval=FALSE}
  git clone https://github.com/username/docker-example.git
  docker pull username/docker-example
  cd docker-example
  docker run --user root -v $(pwd):/home/rstudio/ -p 8787:8787 -e DISABLE_AUTH=true docker-example .
  ```

C) The final option would be for you to generate a [Binder](http://mybinder.org) link to your
container remotely hosted on Docker Hub. For that, you need to create a new `Dockerfile`
in your GitHub repository, and save it in a directory called `.binder`. This new `Dockerfile`
will point to your Docker Hub image under the FROM tag. You can see an example
[`.binder/Dockerfile`](https://github.com/AIMS/docker-example/blob/master/.binder/Dockerfile)
on our GitHub repo. Once this is done, navigate to http://mybinder.org, and paste the link of your
GitHub repository. Binder will generate a launcher badge link that you can then add to your gitHub repo's
`README.md` file, like this

  ```{results='markdown', eval=FALSE}
  [![Launch Rstudio Binder](http://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/username/docker-example/master?urlpath=rstudio)
  ```

Notice that the end part of the link, `?urlpath=rstudio`, is not originally provided by Binder, but
adding it will make sure that an RStudio GUI is triggered on the web browser. All you need to do now is
commit the change to your README and push it. This last option is more laborious, but most definitely
ideal if your collaborators only want to have access to your code.

## Acknowledgements

We would like to thank Drs [Daniel Falster](https://danielfalster.com/) and
[Saras Windecker](https://www.smwindecker.com/) for providing us with a
[great example](https://github.com/dfalster/Westoby_2012_JTB_sapwood_model)
on how to implement this reproducibility framework. Also, we thank the
[rocker](https://github.com/rocker-org/rocker-versioned) and [Binder](http://mybinder.org) teams
for making this much possible.
