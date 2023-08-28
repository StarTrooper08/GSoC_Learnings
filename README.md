## My Experience and Learnings from GSoC 2023(R Project For Statistical Computing)
![R codespace](https://github.com/StarTrooper08/GSoC_Learnings/assets/72031540/8633588f-2cc5-4970-8790-572ab3ebc33e)

### How it started and got to know about the project I'm working on this GSoC'23

I started learning Devops tools and was pretty much known to tools like Docker and Github Actions. One of my Best friends sent me the project description. After reading through the project description I found it was somewhere matching the skills I had and wanted to give it a try. I tried out the task listed below project idea on gsoc project idea page. And discussed the project with respective mentors and that is how it started all.

### Project Info

The project aims to create an environment for open source contributors to easily set up and install R devel source code and let them work on bugs/issues instead of getting tangled in the setup process. This environment is great for beginners who are getting started contributing to the R community. The project uses tools like bash scripting, R lang, and Docker and uses GitHub codespace as a platform to connect all together and create an environment for contributors to contribute to the R code base.

#### Project Repository:
https://github.com/r-devel/r-dev-env/

#### Documentation :
https://contributor.r-project.org/r-dev-env/

#### My Project Mentors :
Heather Turner and James Tripp


### Challenges

1. R Contribution Workflow

The main and important component was the R contribution workflow. And we were setting this up on codespace using docker and bash scripting. The normal R setup steps on the local machine for R studio were mentioned on the R documentation page but here we have performed these steps on Codespace which is a VSCode editor but on a browser so the process of R installation was a little bit different. It was quite challenging for me to try out the R installation process on codespace(VSCode) cause I've never worked around the R source code that much but both of my Mentors helped me here and we were able do the basic R setup on codespace after doing little bit changes in dockerfile and bash commands.

2. Changing the terminal path to use the custom-installed R version

For this, we have to write a bash script that can switch between the default R version which is present in the Docker image and the custom R version one can build inside the codespace. At the start, we used the devcontainer.json file which is needed to build the R development environment inside codespace. But it was keeping the paths fixed and we were unable to switch between the default R version which we have inside the docker image and custom R which we can build inside the codespace.

We then tried to manipulate the settings.json file which sets the R terminal paths using the Linux jq package which is used to do manipulation of JSON with bash script. And it worked for us!!!

3. Optimizing the Docker Image.

The R development environment is based on Docker Image. For now, we are focused on using it with Github Codespace. Since it is based on docker, the image size is quite large and it takes time to build and load the codespace for the first time.
But after building the docker image before then mentioning the docker image registry link inside the devcontainer.json it reduced the time to load the codespace and became quite fast.
But one thing to note is that we can also run the R development environment locally using docker. But there it takes more time to load. To reduce the environment loading time, I thought to use slimtoolkit which is a docker image size-reducing tool and also a security check tool similar to docker scout.
To reduce the docker image size, we have to figure out which Linux packages and tools we need to run the R development environment properly locally.

### Future Goals

The R development environment project is still in the development stage and a few things we are working on are solidifying the R terminal path switching, and polishing R dev env documentation. Optimizing the docker image to use locally will load the R development environment much more quickly than it is now.

### What did I learn during this GSoC Period?

During the GSoC period, I learned a lot of new things as a developer. The first thing I got used to was bash scripting earlier I use to know basic commands and bundle them inside the shell script file.

But for this project, I worked with complex bash scripting and got to know the possibilities it offers.

Never thought how many things we can automate using Github actions. After building Github actions for mkdocs build with guidance from James Tripp and slimtoolkit Github actions for building and optimizing docker image. This Learning will help me as an Emerging Devops Engineer.

Moreover, I got to know about tools like 'hypothes' which can be useful for Open Source dev who works with documentation.

### Links to Work

- R Development Environment - https://github.com/r-devel/r-dev-env/

- Slimtoolkit Actions Workflow - https://github.com/StarTrooper08/SlimtoolkitActions
