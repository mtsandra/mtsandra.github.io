---
layout: post
title:  "Building My Website with GitHub Pages"
date:   2022-10-11
description: My website setup process with bug fix
tags: tech-tutorial
categories: tutorial
comments: false
---

I've always wanted a neat little personal website to consolidate my writings and personal projects (both artistic and technical) in one place. Ideally, it would be minimalistic and not too complicated to maintain. I'm also cheap, so I don't want to pay for website hosting. All these factors made Github Pages the perfect site for me – free with a little bit of work. 

Turns out I'm a little bit particular when it comes to the website aesthetics. My first choice is [HardCandy](https://github.com/xukimseven/HardCandy-Jekyll) because of Marceline from Adventure Time (I love BMO!) used in the demo. But the template is a little outdated and I couldn't get it to work. Then I came across the [al-folio](https://github.com/alshedivat/al-folio) template, which is what my current website is built off of. 

I have limited experience with Jekyll before, so I did this weirdly satisfying step by step [tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/) on the Jekyll official website to help me understand how Jekyll sites work. I found the below command really helpful, it allows your work-in-progress website hosted on the local server to reload automatically as you are making updates. Note that updates made to the _config.yml do not refresh until you restart your server. But for other updates, it refreshes automatically.
```
jekyll serve --livereload
```

Once it worked fine on my local environment, I decided to deploy it to my github.io server to make sure this template works before I'd spend way too much time invested in it. Morphe's Law kicked in again, GitHub was throwing me an error in the build workflow :sweat_smile:

```
github-pages 227 | Error: Liquid syntax error (line 5): Unknown tag 'twitter'
```
I tried a couple of solutions on al-folio's GitHub repository (shoutout to them for keeping it active and alive), but none of them worked. I even tried the manual deployment option listed in their Readme file. Frustrated, I forced myself to take a magic walk break. I love walk breaks, the problems I'm stuck on are usually resolved magically after my walk break. Anyways, it's because the deploy file has the source branch as ```master```, but GitHub's default branch has been renamed to ```main```. I changed all the "master" in ```./bin/deploy``` to "main".

I also want to point out that it is essential to make sure that your GitHub Pages is deployed from ```gh-pages``` and not ```main```. You can change this in the Settings page of your github.io repository. 

After both things are done, I pushed the updated code to my github.io repository, and it worked!! :heart_eyes: Now I have a working personal website!'

#### Resources

For a step by step set up using the al-folio template, please refer to their README file [here](https://github.com/alshedivat/al-folio). I found it to be really simple and effective. This blog post is to document my first encounter with Jekyll and I'm hoping it could help anyone who is having trouble with the same build error. 

See you next time! :laughing:

-ATM
