---
layout: post
title: "Posting to Twitter/Mastodon with Github Actions"
categories: github
---

# Introduction

So this blog site is relativly new, as you well know, and when I put it together, I wanted a way to make it easier for me to make posts. In the past I have used Wordpress, but I have found that the industry is moving more towards using static sites vs. dynamic sites. Between ease of use/deployment and the inherrant security issues with a dynamic site, it makes sense. Not that Wordpress is by any means a bad CMS to use, I just didn't want the headaches of hosting it here and constantly maintaining it to patch holes. So I wanted to try working with a static generator. I did some research and landed on Jekyll and Github Pages. That really peeked my interest because of:

* learning a new skill
* being able to incorporate my love of using Git/Github in my deployment of the site
* the geek factor

Since I am using Github and have a CI/CD pipeline (Github Action) already setup to automagically deploy my blog site on a push to the website repo, I also wanted to setup a workflow that would post a Tweet to Twitter and a Toot to Mastodon at the same time. It would save me a few minutes each time when I made a new post to the blog. And.... you know.... geek factor.

So I thought I would share how I did it.

This method will work for any repo you have, but you may have to tweak the workflow for your specific use case.

## Assumptions

In relation to the use of the workflow file I am going to share:

* You have a repo you want to post about
* You are familiar with Jekyll and it's directory structure
* You are familiar with YAML
* You are familiar with adding ```SECRETS``` to your Github Repo
* You are familiar with creating a ```workflow.yml``` file in Github Actions

## Obtaining API Keys

In order for this to work, you will need to obtain some API keys from Twitter and Mastodon.

For Twitter, you will need 4 Keys. A walkthrough on how to get them is here: [https://developer.twitter.com/en/docs/twitter-api/getting-started/getting-access-to-the-twitter-api](https://developer.twitter.com/en/docs/twitter-api/getting-started/getting-access-to-the-twitter-api)

* ```TWITTER_CONSUMER_API_KEY``` and ```TWITTER_CONSUMER_API_SECRET``` are the API Key and API Secret respectfully.
* ```TWITTER_ACCESS_TOKEN``` and ```TWITTER_ACCESS_TOKEN_SECRET``` are the access Token and Access Token Secret respectfully.

For Mastodon, you just need the Application Access Token.
* Go to your Mastodon Instance and Click the Account Settings icon (Gear Icon) for your account
* Find the ```Development``` Section.
* Click ```New Application```
* Give it a name (I called mine "New Post to Blog")
* Make sure to add the ```write:statuses``` scope in the check boxes below
* Save it.
* You should then see 3 keys. You will need the ```Your Access Token``` one.

You will then need to put all of these keys into seperate secrets in your Github repo. I would name them the same as in the YAML file so you can just copy and paste the YAML file, with minor tweaks of course.

## The Workflow YAML file

Now we can create a new workflow YAML file for our pipeline. You should already be familiar with creating an Action from scratch, so all you need to do is copy and paste the below YAML into a new one.

Here is the workflow YAML file I finally ended up with:

```yaml
name: Send a Tweet
on:
  push:
    paths:
        - '_posts/**'
jobs:
  tweet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: send-twitter
        uses: infraway/tweet-action@v1.0.1
        with:
          status: "NEW POST: ${{ github.event.commits[0].message }} https://n8acl.github.io"
          api_key: ${{ secrets.TWITTER_CONSUMER_API_KEY }}
          api_key_secret: ${{ secrets.TWITTER_CONSUMER_API_SECRET }}
          access_token: ${{ secrets.TWITTER_ACCESS_TOKEN }}
          access_token_secret: ${{ secrets.TWITTER_ACCESS_TOKEN_SECRET }}
      - name: send-mastodon
        uses: rzr/fediverse-action@v0.0.6
        with:
          access-token: ${{ secrets.MASTODON_ACCESS_TOKEN }}
          host: "mastodon.radio"
          message: "NEW POST: ${{ github.event.commits[0].message }} https://n8acl.github.io"
```

### Breakdown

So let's break this YAML file down.

```yaml
name: Send a Tweet
on:
  push:
    paths:
    - '_posts/**'
```

First I give the job a name and then I only want it to run when there is a push to the ```_posts``` folder in the repo, basically when I add a new post file. That is what I am saying here. I am filtering the pushes on that folder. That way if I make an update to a static page like a contact page or an about page, I don't want those commits posts to the Micro-Blogging Services.

```yaml
jobs:
  tweet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: send-twitter
        uses: infraway/tweet-action@v1.0.1
         with:
          status: "NEW POST: ${{ github.event.commits[0].message }} https://n8acl.github.io"
          api_key: ${{ secrets.TWITTER_CONSUMER_API_KEY }}
          api_key_secret: ${{ secrets.TWITTER_CONSUMER_API_SECRET }}
          access_token: ${{ secrets.TWITTER_ACCESS_TOKEN }}
          access_token_secret: ${{ secrets.TWITTER_ACCESS_TOKEN_SECRET }}
```

Here I am defining the job itself. This first step is where I am sending a message to Twitter that there is a new post.

```yaml
      - name: send-mastodon
        uses: rzr/fediverse-action@v0.0.6
        with:
          access-token: ${{ secrets.MASTODON_ACCESS_TOKEN }}
          host: "mastodon.radio"
          message: "NEW POST: ${{ github.event.commits[0].message }} https://n8acl.github.io"
```

This last part does the same as the Twitter part, but just posts it to Mastodon. Make sure that you set the ```host:``` line to your Mastodon Instance. Otherwise it will default to Mastodon.social.

### How it works

When I commit a new post to the repo, I need to make sure that I use a commit message that is the title of the post. So for this post for example I would use "Posting to Twitter-Mastodon with Github Actions - 06212022". This would generate a status post of "NEW POST: Posting to Twitter-Mastodon with Github Actions - 06212022 https://n8acl.github.io" on Twitter and Mastodon.

Now my followers on those platforms will know when I have made a new post.

Keep in mind you will have to change your status to fit your use case.

# Wrap Up

And there you have it. Now when I make a new post, my Github Pipeline with send a tweet that there is a new post on the website and automatically deploy the actual post to my website. 

This would work with any Repo you want to share about on Social Media even. You would just need to modify it to match your use case and what you want sent.

I am planning on using this on a number of my other repos as well to make deployment just a little less painful when I update other projects I am working on.

# Sources

While I am not using these exactly how they are defined in these articles, these were my jumping off point on getting this all configured.

Tweeting New GitHub Pages Posts from GitHub Actions - Dave Brock: [https://www.daveabrock.com/2020/04/19/posting-to-twitter-from-gh-actions/](https://www.daveabrock.com/2020/04/19/posting-to-twitter-from-gh-actions/)
fediverse-action repo: [https://github.com/rzr/fediverse-action](https://github.com/rzr/fediverse-action)