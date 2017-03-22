+++
#prev = "/prev/path"
weight = 5
title = "Blogging with GitHub and Hugo"
date = "2017-03-22T23:18:03Z"
#toc = true
#next = "/next/path"

+++


As a fairly late comer at the blogging party, I benefit greatly from others in gaining knowledge to create my own site.

The main documentation I followed while creating this blog is [Amber Thomas'](https://proquestionasker.github.io/) post [Making a website using Blogdown, Hugo and GitHub pages](https://proquestionasker.github.io/blog/Making_Site/). It is a well written tutorial that expands on all aspects of the process. It is well worth a read if you are a beginner on Github, website building and theming. I just find it handy that everything is in one place, and often with good reference from other sources.

My attempt at making this blog, however, started further along still with more development on Hugo's documentation on working with Github, i.e. documented by [Jente Hidskes](https://hjdskes.github.io/) in the [updated post](https://hjdskes.github.io/blog/update-deploying-hugo-on-personal-gh-pages/) on the original one that Amber referred to. While there is no need to repeat what other people have explained eloquently, I try to point out catchas and slight errors I found in the reference, in the hope that whoever ends up reading this can avoid the many repeats I went through.

1. The first thing to keep in mind is that, while the process seems complicated at first, pretty much everything is reversible (except any existing blog post content if it's included inside the website folders, save them somewhere else before you are happy with the website itself). Be brave and work through the steps, if there is a messed-up, the worst possible thing is that you delete the remote and the local repo and start again. Be fearless.

2. While setting up Github to work with the local repo, opt straight for the SSH method, no need to dwell on HTTPS. That way you can avoid having to go back and change it when starting to run the Shell script. Another benefit is of course saving your entering the password every time you sync the remote and local repos.

3. The `blogdown` package is brilliant at simplifying steps of generating the basic website, these simple lines below give you a complete set of skeleton website files:
	```R
	install.packages('devtools')
	devtools::install_github("rstudio/blogdown")
	library(blogdown)
	install_hugo()
	new_site()
	```
	Just a couple of things to watch out for:
	* After hugo is installed using the `R` command (or indeed in the CLI), make sure that the installation directory is included in the operating system environment variable path. If it is not, the subsequent script for website deployment will not work (as it calls the `hugo` command directly). Follow [these](https://www.schrodinger.com/kb/1842) to set up the path variable.
	* `new_site()` function is a wrapper for `hugo` command that generates the website files, it also includes theme downloading by default. This problem could be unique to me, but there were a few times the function downloaded the theme package and named the folder incorrectly (appending `-master`  at the end at random), which causes the subsequent steps of the function to fail to find the theme. The solution is simply, clear the folders and run the function again! (And unsurprisingly running the function `install_theme()` can also have this issue, as it is called inside `new_site()`, but this doesn't affect any automated process so is easy to rectify.)

4. As [confirmed with Jente](https://github.com/Hjdskes/hjdskes.github.io/pull/1), [`setup.sh`](https://hjdskes.github.io/blog/update-deploying-hugo-on-personal-gh-pages/) needs a modification: add `add` to the final line between `worktree` and `-B`. So the complete script is:
	```shell
	#!/usr/bin/env bash

	# This script does the required work to set up your personal GitHub Pages
	# repository for deployment using Hugo. Run this script only once -- when the
	# setup has been done, run the `deploy.sh` script to deploy changes and update
	# your website. See
	# https://hjdskes.github.io/blog/update-deploying-hugo-on-personal-github-pages/
	# for more information.

	# Further to Jente's original script, 'add' was added to the final line between 'worktree' and '-B' - https://xinye1.github.io/blog/

	# Name of the branch containing the Hugo source files.
	SOURCE=hugo

	msg() {
	printf "\033[1;32m :: %s\n\033[0m" "$1"
	}

	msg "Adding the \`public\` folder to .gitignore"
	echo "public" >> .gitignore

	msg "Deleting the \`master\` branch"
	git branch -D master
	git push origin --delete master

	msg "Creating an empty, orphaned \`master\` branch"
	git checkout --orphan master
	git reset --hard
	git commit --allow-empty -m "Initial commit on master branch"
	git push origin master
	git checkout $SOURCE

	msg "Adding the master branch into the \`public\` folder"
	rm -rf public
	git worktree add -B master public origin/master
	```
5. To-do: customise the theme to include syntax highlighting​.