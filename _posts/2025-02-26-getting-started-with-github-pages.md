---
layout: post
title: "TEST - Getting Started with GitHub Pages"
date: 2025-02-26
categories: [Web Development, GitHub]
featured_image: /assets/images/benchmark-blue.webp
read_time: 5
excerpt_separator: <!--more-->
published: false
---

GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files straight from a repository on GitHub, optionally runs the files through a build process, and publishes a website.

<!--more-->
  

## What is GitHub Pages?

GitHub Pages allows you to host static websites directly from a GitHub repository. It's free to use and is a great platform for personal websites, project documentation, or blogs.
  

## Setting Up Your GitHub Pages Site

1. Create a new repository on GitHub named `username.github.io`, where `username` is your GitHub username.
2. Clone the repository to your local machine.
3. Add your HTML, CSS, and JavaScript files to the repository.
4. Commit and push your changes.
5. Your site will be available at `https://username.github.io`.

## Using Jekyll with GitHub Pages

GitHub Pages has built-in support for Jekyll, a static site generator. This makes it easy to create a blog or website without the need for a database or server-side code.

### Installing Jekyll Locally

To install Jekyll locally, you'll need Ruby and RubyGems installed. Then, you can install Jekyll with:

```bash
gem install jekyll bundler
```

<p align="center">
  <img src="/assets/images/benchmark-blue.webp" alt="Centered Image" width="768">
</p>

### Creating a New Jekyll Site

To create a new Jekyll site, run:

```bash
jekyll new my-blog
cd my-blog
bundle exec jekyll serve
```

Your site will be available at `http://localhost:4000`.
  

## Conclusion

GitHub Pages is a fantastic way to host your static websites for free. Combined with Jekyll, it provides a powerful platform for creating and maintaining blogs and documentation sites.
