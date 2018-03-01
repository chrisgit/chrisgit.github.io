---
title: "Docker: Standardised Tooling"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Docker
  - DevOps
  - Tools
---
Setting up a machine for Development or Operations can be time consuming; there are core packages to install and configure; conflicts in libraries that may need resolving, you might even be restricted to installing specific versions of software.

If you are disciplined you might destroy and rebuild your machine on a regular basis and use scripts or a provisioning tool to speed up the process.
<!--more-->
The longer I play with Docker the more opportunities for experimentation come up; such as could you run an operating systems that does just one thing and what if that thing was Docker? Sure, there are already container specific operating systems like Rancher OS and CoreOS but these are specialised.

## What are we solving?
The first thing to consider is the problem, the installation of software or the "standard installation". The company I work for are using Terraform to build their infrastructure; it started with small groups of the engineering teams using Terraform and has rapidly expanded to other technical areas of the business. Now we want to formalise the development process so have setup a community of interest and are looking to incorporate CI/CD.

To add CI/CD to the process we need build slaves, these will need to be installed with Terraform and labelled "infrastructure", simple right? Unfortunately as the technology spread each team took the opportunity to use the latest version of Terraform and took advantages of the latest features.

The solution is that we put the latest version of Terraform on the build slaves and just upgrade everyone, job done! In reality it's not that simple, sometimes the software has breaking changes, other times we may not want to use the latest version. Additionally it raises the question, can different versions of the software sit side by side and if so how many versions do we keep? Are there likely to be any shared library issues?

## Docker for tooling
We'd seen a similar spread of technology before with Chef and inpite of having a Wiki page, community of interest and regular news letters the first question new users ask is "what version do I use?", the answer was different depending on who you asked and what they had installed as their "stable version", the dreaded works on my machine syndrome.

In order to create standardised tooling using Docker I picked Chef as my first experiment; because I know it quite well and know what to expect of the underlying Ruby environment should I need to debug it.

Downloading the latest [ChefDK image](https://hub.docker.com/r/chef/chefdk/) I tried a few simple experiments with Docker run, such as getting the version of Ruby, ChefDK and eventually attempting a converge. The extract below expects the image tag to be stored in ${CHEFDK_VERSION}
```
docker run -i --rm -v \$(pwd):/chef-repo chef/chefdk:\${CHEFDK_VERSION}
```

Rather than typing docker run I created an alias named chef.
```
alias chef="docker run -i --rm -v \$(pwd):/chef-repo chef/chefdk:\${CHEFDK_VERSION}"
```

The alias appeared to do everything I wanted so moved on to a basic build script
```
export CHEFDK_VERSION=0.15.15

set -x
shopt -s expand_aliases
alias chef="docker run -i --rm -v \$(pwd):/chef-repo chef/chefdk:\${CHEFDK_VERSION}"

COOKBOOK_NAME=${JOB_NAME##*/}
COOKBOOK_BUILD_NUMBER=${BUILD_NUMBER}
CHANGE_LOG_PATH=$(dirname $0)

cd ${WORKSPACE}

chef /bin/bash -c "export COOKBOOK_NAME=${COOKBOOK_NAME}; \
                              export COOKBOOK_BUILD_NUMBER=${COOKBOOK_BUILD_NUMBER}; \
                              export CHANGE_LOG_PATH=${CHANGE_LOG_PATH}; \
                              ruby /chef-repo/scripts/build.rb -c $COOKBOOK_NAME -b $COOKBOOK_BUILD_NUMBER -w /chef-repo/cookbooks/$COOKBOOK_NAME"
```
NB: The files that are being executed by the tools in the Docker container are not stored in the images themselves but held locally on the host operating system and are exposed to the containers by mounting the folders as volumes.

Basics covered, what about Terraform? Terraform has a few more command line switches that developers are accustomed to using so a script using an alias was not possible; for Terraform we have streamlined the developer experience in two ways
- Create a VirtualBox VM running a patched up to date CentOS running Docker
- Provided a script which is available in the path to run a Terraform container

The rough and ready first draft of the script is below
```
#!/bin/bash
# Get the command line switches
CMD_LINE=${@:1}

# Setup paths and folders
DOCKER_FOLDER="/scripts/environments/..."

# Environment variables stored in a file - retrieval of AWS keys
cat << EOF > ~/terraform_environment
AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION
EOF

# Run terraform
docker run -it --volume=/vagrant/terraform9:/scripts -w=$DOCKER_FOLDER --entrypoint="/bin/ash" --env-file ~/terraform_environment hashicorp/terraform:$TERRAFORM_VERSION -c "terraform $CMD_LINE"
```
From the users perpective is looks and behaves in almost the same way if Terraform was installed locally.

## Verdict
It's still early days but it is simple for us to use different versions of our tooling as we can download different Docker images and have less clutter on our host machines.