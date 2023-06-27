---
title: Building This Blog
date: 2023-05-05 12:00:00 -0800 
categories: [Guide]
tags: [jekyll,ubuntu,github,blog]
img_path: /assets/img/posts/building-this-blog
--- 
![HTML Code](/florian-olivo-unsplash.jpg)

# Hello and Welcome

If you're reading this then I finally figured out how to build a website, or at least got lucky enough with copy and pasting.

The purpose of this site is to showcase projects that I'm working on, post step-by-step guides on self-hosted/homelab applications, and archive documentation for personal and public use. 

This post will be a bit meta in the sense that it's a post showing how I made the site, hosted it on Github, and uploaded this post. Maybe you want to build your own blog, and here is a guide to show you how.

## Getting Started 

### Requirements

- Github Account
- Jekyll and it's dependencies
- IDE - I use VSCode

### Jekyll Install

My daily driver's distro is Ubuntu, so the instructions will reflect that, but it shouldn't be too difficult to apply to your own specific OS.

Install Ruby and other prerequisites:
```bash
sudo apt-get install ruby-full build-essential zlib1g-dev git
```

Avoid installing RubyGems packages (called gems) as the root user. Instead, set up a gem installation directory for your user account. The following commands will add environment variables to your ~/.bashrc or ~/.zshrc (if using zsh as your shell) file to configure the gem installation path:
```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Finally, install Jekyll and Bundler:
```bash
gem install jekyll bundler
```

### VSCode Install on Ubuntu 

Download .deb at <https://code.visualstudio.com/download>

Open a terminal and run the following commands:
```bash
cd Downloads/   # change directory to default Downloads folder
sudo apt install ./FILE.deb   # installs .deb file
```

### Find a theme to build upon 

These are a few places to find free Jekyll themes and templates:

 - <https://github.com/topics/jekyll-theme>
 - <https://jekyllthemes.io/free>
 - <https://jamstackthemes.dev/#ssg=jekyll>

## Building From a Jekyll Theme

The theme that I chose to build this site from is Chirpy .It seemed like a pretty clean and modern site, has pretty solid documentation, and through the process of getting all of this set up this theme gave me the least issues. You can find it here: 

 - <https://github.com/cotes2020/chirpy-starter>
            
            
Make sure you're logged into your Github, navigate to the desired theme repo, select _Use This Template_ or _Fork_

![Github Code Button](/BuildingThisBlog1.png)

Name repository with specific convention      

- [YOUR_GITHUB_USERNAME].github.io 


Make sure the repository's visibility is set to _Public_

Click save
                
### Setup to edit remotely (edit in VSCode)

On your repo select Code, and copy the URL 

![Github Clone Button](/BuildingThisBlog2.png)

Open VSCode and open the command palette (F1 or Shift+Ctrl+P)
Run _Git: Clone_

![VSCode Git: Clone](/BuildingThisBlog3.png)

Paste the URL from your repository

![VSCode Clone From URL](/BuildingThisBlog4.png)

Select where you would like to save the cloned repository

![VSCode Save Repo](/BuildingThisBlog5.png)

Once the repository has finished downloading click _Open_

![VSCode Open Repo](/BuildingThisBlog6.png)

With the cloned repository opened up in VSCode, open up the terminal inside VSCode (View -> Terminal or Ctrl+`)

```bash
pwd                   # print working directory
cd /PATH/TO/REPO      # change directory to location of cloned repo
bundle                # installs dependencies
bundle exec jekyll s  # launches the site on a local server
```

Now the site is running locally on your machine, just open a browser and go to <http://127.0.0.1:4000/> to view the site. This is useful for once you start making changes to the site such as posts, layouts, themes, etc. and want to see how changes look before committing to pushing those changes.

## Customize Site 

Open __config.yml_ in VSCode

![VSCode Sidebar](/BuildingThisBlog7.png)

Make changes to match your info (Title, Tagline, Description, URL, Socials, etc.) 

URL: https://[GITHUB_USER].github.io

Save __config.yml_ 

 - Changes made will reflect to site once saved and site is refreshed 
 - If changes aren't reflected, stop and restart server by running the following commands in the VSCode terminal 
```bash
Ctrl+C                  # press Ctrl+C to stop server
bundle exec jekyll s    # starts local server
```

## Creating Content - Posts

The theme that you use may or may not include documentation on adding content to your site that could be specific to your theme, it's worth looking into. I've posted a link to a guide that my theme's author has put together, it may be of some benefit to you.

 - <https://chirpy.cotes.page/posts/write-a-new-post/>

Create a new file in the format of _YYYY-MM-DD-TITLE.md_ and put it in the __posts_ folder of your cloned repository.

At the top of the file paste in the following and editing to your liking:
```
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORY, SUB_CATEGORY]
tags: [TAG]     # TAG names should always be lowercase
---
```

After the YAML block has been added to the top of your post, you are free to write until your heart's content. Just don't forget to save periodically.

### Adding images to posts
                
In your cloned repo, folders beginning with an underscore won't be copied to the build destination. So, the most common way is to store images is to add them to a _assets/img_ folder. It's also worth keeping your images organized by creating a folder inside the _assets/img_ folder named _posts_ and keeping your images in folders named after their respective posts.

![Image Folders](/BuildingThisBlog8.png)

When a post contains many images, it will be a time-consuming task to repeatedly define the path of the images. To solve this, we can define this path in the YAML block of the post:
```
---
img_path: /PATH/TO/IMAGES
---
```
And then, the image source of Markdown can write the file name directly:
```
![The flower](flower.png)
```

## Deploy to Github using Github Actions

Once your post has been completed, you'll push the changes to Github. First you'll make the required changes for enabling Github Actions.

On the main page of your repository, select the _Settings_ tab, then click _Pages_ in the left navigation bar. Then under the _Build and Deployment_ section click the dropdown menu under _Source_ and select _Github Actions._

![Github Actions](/BuildingThisBlog9.png)

In your VSCode terminal run the following:
```bash
git status   # Check git status to check for changes
git add .    # Stage changes 
git status   # Check git status, should show changes to be committed
git commit -m "feat(post): Building This Blog" # Commit changes and add comment 
git push     # Push changes up to Github
```

After the _git push_ has been ran, wait a couple of minutes for Github to finish building the site and you should get a green check on the main page of your repository:

![Github Commit Success](/BuildingThisBlog10.png)

And that's it! You've now hosted and published your site. Anytime you want to add more posts or make any changes to your site, just make sure all files are saved and re-run that last set of commands; _git status, add, status, commit, push._

## Thank you and I hope you find this useful!

## References 

- <https://jekyllrb.com/>
- <https://www.markdownguide.org/>
- <https://html5up.net/>
- <https://unsplash.com/>