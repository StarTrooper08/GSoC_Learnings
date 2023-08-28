# My Experience and Learnings from GSoC 2023(R Project For Statistical Computing)
![R codespace](https://github.com/StarTrooper08/GSoC_Learnings/assets/72031540/8633588f-2cc5-4970-8790-572ab3ebc33e)

### How it started and got to know about the project I'm working on this GSoC'23

I embarked on a journey to learn DevOps tools and already had a decent understanding of tools like Docker and GitHub Actions. It was during this time that one of my best friends shared a project description with me. Upon reviewing the project details, I noticed a strong alignment between the required skills and my existing capabilities, prompting me to consider taking on the challenge. To gauge my fit for the project, I tackled the tasks listed within the project idea on the GSoC project page. Subsequently, I engaged in discussions with the mentors associated with the project. This marked the inception of the journey that led me to where I am today.

### Project Info

The project's objective is to establish an environment tailored for open-source contributors, simplifying the process of configuring and installing R devel source code. This streamlined setup facilitates their engagement in addressing bugs and issues, alleviating the challenges typically associated with the initial setup. Notably beneficial for newcomers embarking on contributions to the R project, this environment harnesses tools such as bash scripting, R programming, and Docker. The integration of GitHub Codespaces serves as the cohesive platform, uniting these elements to provide contributors with a conducive space for actively participating in the enhancement of the R codebase.

#### Project Repository:
[Project- R-Dev-Env](https://github.com/r-devel/r-dev-env/)

#### Documentation :
[Docs- R-Dev-Env](https://contributor.r-project.org/r-dev-env/)

#### My Project Mentors :
Heather Turner and James Tripp


### Challenges

**1. R Contribution Workflow**

The central part of our project was the R contribution workflow, a crucial component. We set this up in Codespaces using Docker and bash scripting. Normally, the steps to build R on a local Ubuntu machine were explained in the R Development Guide. However, in Codespaces, which is a browser-based version of VSCode, the process was a bit different due to its unique environment. For me, it was a challenge to adapt the R installation process to Codespaces because I hadn't worked extensively with R source code before. Thankfully, both of my mentors provided valuable assistance. Together, we managed to create a simple contributor workflow: building R, making small code changes, and seeing those changes in Codespaces. This required tweaking the dockerfile and using specific bash commands.

**2. Changing the terminal path to use the custom-installed R version**

For this, we have to write a bash script that can switch between the default R version which is present in the Docker image and the custom R version one can build inside the codespace. At the start, we used the devcontainer.json file which is needed to build the R development environment inside codespace. But it was keeping the paths fixed and we were unable to switch between the default R version which we have inside the docker image and custom R which we can build inside the codespace.

We then tried to manipulate the settings.json file which sets the R terminal paths using the Linux jq package which is used to do manipulation of JSON with bash script. And it worked for us!!!

**3. Optimizing the Docker Image**

The R development environment is based on Docker Image. For now, we are focused on using it with Github Codespace. Since it is based on docker, the image size is quite large and it takes time to build and load the codespace for the first time.
But after building the docker image before then mentioning the docker image registry link inside the devcontainer.json it reduced the time to load the codespace and became quite fast.
But one thing to note is that we can also run the R development environment locally using docker. But there it takes more time to load. To reduce the environment loading time, I thought to use slimtoolkit which is a docker image size-reducing tool and also a security check tool similar to docker scout.
To reduce the docker image size, we have to figure out which Linux packages and tools we need to run the R development environment properly locally.

### Implementations for R-Dev-Env v0.1 and upcoming v0.2 :
**1. Added Path Env Var $BUILDDIR and $TOP_SRCDIR to Dockerfile**

The env var is crucial implementation in r-dev-env project since these both path var help us source code and build version in 2 different directory.
We first tried out adding these both env variable path inside the bash script and running the script when the codespace is created since these 2 variable needed to be present all the time. We mentioned them inside dockerfile.

Dockerfile Snippet for env var 
```Dockerfile
ENV BUILDDIR='/workspaces/r-dev-env/build'
ENV TOP_SRCDIR='/workspaces/r-dev-env/svn'
```

###### PR related to this implementation :
[PR for Path Env Var](https://github.com/r-devel/r-dev-env/pull/11)
[PR for Path Env Var update](https://github.com/r-devel/r-dev-env/pull/69)

**2. R contribution Workflow**

The R contribution Workflow is also one of the implementation needed to let users build R version on R dev env.
One can install and build R version with or without recommended packages.
This process is done manually by users by following the steps given R dev env docs. In further release, this thing can be automated.
[Documentation](https://contributor.r-project.org/r-dev-env/tutorials/building_r/) to install and build R on R dev env

###### PR related to this implementation :
[R contribution workflow docs](https://github.com/r-devel/r-dev-env/pull/64/files)


**3. Mkdocs Github Actions**

The Mkdocs is used to document the usage of R development Environment project. For that we wanted to automate the build and publish to github pages process. So we build the github actions workflow for this process.

YML file Snippet for mkdocs Github Actions workflow
```YML
name: Build MKDocs website

on:
  push:
    branches: ["devel"]

jobs:
  build-and-deploy:
    if: ${{ contains(github.event.head_commit.message, 'Build website') }}
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.x 

      - name: Install dependencies
        run: pip install mkdocs

      - name: Build MkDocs
        run: mkdocs build

      - name: Deploy to GitHub Pages
        run: |
           mkdocs gh-deploy --force
           git push origin gh-pages
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

###### PR related to this implementation :
[PR for the Mkdocs build and publish process GH Actions](https://github.com/r-devel/r-dev-env/pull/64)

**4. Building Multiple R version**

The R Development Environment let us install and build multiple R version. We have tested and documentated the installing and building multiple R versions with different name and it is based on the R contribution workflow setup
You can find the docs for Building Multiple R version [here](https://contributor.r-project.org/r-dev-env/)

###### PR related to this implementation :
[PR for Building Multiple R version](https://github.com/r-devel/r-dev-env/pull/74)

**5. Switch between R terminal path**

The R development environment comes with R v4.3.1 version by default bundled up. But the main motive of the R dev env project is to let user install and build any different R version and do the code contribution by fixing the bugs/issues.
And one can install and build R version using the contribution workflow docs and multiple R build.
But the issue here was switching R terminal path between the multiple R versions properly.
To do so we need to manipulate the settings.json for R extension inside the codespace. In the beginning we directly updated the devcontainer.json and made the required changes but it used to update the path but doesn't use to switch back to default R version from the custom R build.
So we written bash script to switch between default R version of R dev env and custom R build.

Bash Script snippet 
```bash
#!/bin/bash

# Path to the settings.json file
settings_file_path="/home/vscode/.vscode-remote/data/Machine/settings.json"


# Read the existing JSON content
settings_data=$(cat "$settings_file_path")

echo "Do you want to switch rterm path to custom R build / default R build? (type C or D)"
read option

if [ "$option" = "C" ]; then
    updated_settings_data=$(echo "$settings_data" | jq '."r.rterm.linux"="/workspaces/r-dev-env/bin/R/bin/R"')
elif [ "$option" = "D" ]; then
    updated_settings_data=$(echo "$settings_data" | jq '."r.rterm.linux"="/usr/bin/R"')
else
    echo "Invalid option selected."
    exit 1
fi

echo "$updated_settings_data" > "$settings_file_path"

```

The above script just let us switch between default R version and one specifed custom R build. So to switch between default and multiple R build version, we have written bash script to do so. Its currently in testing and will be released with v0.2 of R Development Environment.

Multiple R terminal switch
```bash
#!/bin/bash

# Specify the parent directory
parent_dir="$BUILDDIR"

# Path to the settings.json file
settings_file_path="/home/vscode/.vscode-remote/data/Machine/settings.json"

echo "Do you want to switch rterm path to custom R build / default R build? (type C or D)"
read option

if [ "$option" = "C" ]; then
    # Check if the parent directory exists
    if [ -d "$parent_dir" ]; then
        echo "Subdirectories in $parent_dir:"
        
        # Create an array to store subdirectory names
        subdirs=()
        
        # Loop through subdirectories and populate the array
        for dir in "$parent_dir"/*; do
            if [ -d "$dir" ]; then
                subdir=$(basename "$dir")
                subdirs+=("$subdir")
                echo "${#subdirs[@]}. $subdir"
            fi
        done
        
        if [ "${#subdirs[@]}" -gt 0 ]; then
            read -p "Enter the number of the subdirectory to switch to: " choice
            
            # Check if the choice is valid
            if [ "$choice" -gt 0 ] && [ "$choice" -le "${#subdirs[@]}" ]; then
                chosen_subdir="${subdirs[$((choice - 1))]}"
                cd "$parent_dir/$chosen_subdir" || exit
                echo "Switched to subdirectory: $chosen_subdir"
                
                # Update the settings.json with the chosen subdirectory path
                updated_settings_data=$(cat "$settings_file_path" | jq --arg subdir "$parent_dir/$chosen_subdir/bin/R" '."r.rterm.linux"=$subdir')
                echo "$updated_settings_data" > "$settings_file_path"
                
                # Start an interactive shell in the chosen subdirectory
                bash
            else
                echo "Invalid choice."
            fi
        else
            echo "No subdirectories found."
        fi
    else
        echo "Parent directory does not exist."
    fi
elif [ "$option" = "D" ]; then
    updated_settings_data=$(cat "$settings_file_path" | jq '."r.rterm.linux"="/usr/bin/R"')
    echo "$updated_settings_data" > "$settings_file_path"
else
    echo "Invalid option selected."
    exit 1
fi
```
###### PR related to this implementation :

[PR for R term switch](https://github.com/r-devel/r-dev-env/pull/68)
[PR for multiple R term switch](https://github.com/r-devel/r-dev-env/pull/73)

**6. Slimtoolkit Github Actions**

The slimtoolkit GitHub Actions workflow is designed to optimize Docker images using the slimtoolkit tool, subsequently publishing the optimized Docker image to DockerHub. Although the workflow functions as intended, there are ongoing challenges regarding image optimization. Notably, the slimtoolkit process occasionally removes essential packages and libraries that are crucial for the R development environment. The solution involves incorporating the appropriate library paths and subsequently attempting to optimize the Docker image.
However,this task is on hold for next release.

#### Repository for Slimtoolkit Actions :
[Slimtoolkit Actions Workflow](https://github.com/StarTrooper08/SlimtoolkitActions)


### Link to R Development Environment v0.1 Release :
[R Development Environment v0.1 Release](https://github.com/r-devel/r-dev-env/releases/tag/v0.1)


### Future Goals

The R development environment project is still in the development stage and a few things we are working on are solidifying the R terminal path switching, and polishing R dev env documentation. Optimizing the docker image to use locally will load the R development environment much more quickly than it is now. And also automating few task to make R build process easier.

### What did I learn during this GSoC Period? How was my Experience?

During the GSoC period, I learned a lot of new things as a developer. The first thing I got used to was bash scripting; earlier, I used to know basic commands and bundled them inside shell script files.

However, for this project, I worked with complex bash scripting and got to know the multitude of possibilities it offers. I also wrote an R contribution workflow script under the guidance of my mentor, Heather Turner, which, in turn, helped me build the thought process to write bash scripts. I even learned new Linux commands that I had utilized in this project. I had never realized how many tasks could be automated using GitHub Actions and bash scripting. Building GitHub Actions for tasks such as mkdocs build under the guidance of my mentor, James Tripp, and creating slimtoolkit GitHub Actions for building and optimizing Docker images made me realize the efficiency that can be achieved using GitHub Actions as a developer. This learning will undoubtedly assist me in my journey as an emerging DevOps Engineer.

Furthermore, I became acquainted with tools like [hypothes.is](https://web.hypothes.is/), which can be highly beneficial for open-source developers who work with documentation. I also utilized GitHub's Milestone feature, which I had heard of but never used before. Additionally, I explored the GitHub image registry, which allows us to publish Docker images similar to DockerHub. This experience provided me with a taste of open-source collaboration and enhanced my understanding of efficient development practices.
I also had the opportunity to participate in the R Contribution Working Group, where I presented and discussed the R dev env project. This experience was incredibly valuable as I connected and engaged in conversations with fellow R contributors and community members. The chance to interact with other R enthusiasts provided me with invaluable insights and perspectives on R development Environment Project.
I also received feedback from members of the R community who found the R Development Environment Project helpful. This positive response aligns with the intentions that both my mentors, Heather Turner and James Tripp, and I were striving for. The affirmation from fellow community members reinforces the value of our efforts and encourages us to continue contributing meaningfully to the R ecosystem.

Summarizing what I've learned so far:
- Bash scripting
- New linux commands
- Github Actions automation
- Github Milestone
- Github Image Regitry
- Building Docs using Quarto and Mkdocs.
- Using hypothes.is
- Open Source Collaboration


### Thanks

I want to express my sincere gratitude for the incredible opportunity to be a part of Google Summer of Code 2023. Working under the guidance, my mentors Heather Turner and James Tripp, has been immensely valuable, and I'm thankful for your support and mentorship throughout the program. Being a GSoC participant has not only honed my technical skills but has also shown me the power of open-source collaboration. This experience has been truly transformative, and I'm excited to continue contributing to the community. Thank you once again for this amazing journey.

### Links to Work

- [R Development Environment](https://github.com/r-devel/r-dev-env/)
- [Slimtoolkit Actions Workflow](https://github.com/StarTrooper08/SlimtoolkitActions)


