---
title: "Hello Hugo"
date: 2019-09-19T21:11:57+02:00
disqus: false
---

My first post on this blog is dedicated to the blog itself, and more precisely to the technology that i used to get it up and running. 

First, i worked with a few CMS tools in the past, mostly for friends that wanted a quick page on the internet for their wedding, birthdays, etc etc. 
I remember how easy it was to set up a Wordpress website and make it beautiful by buying a 15$ template. I do also remember how painful and frustrating it was to apply changes and especially how slow the site would have become in the short time. 

Since then i avoided using Wordpress as much as i could, choosing to create the html pages and static assets and serving them via an nginx server. THat requires much more effort, but at the end it really pays off! 

Recently i found on HN many articles talking about [Hugo](https://gohugo.io/), a static site generator written in Go. I couldn't wait to try it and this blog was the pefect fit for that. 

My idea was to use Hugo in order to create a static website, hosting it on [Github Pages](https://pages.github.com/) and redirecting a 7$ domain bough on [Dynadot](https://www.dynadot.com/) to it, sounds easy, let's give it a try! 

First you will need to install Go:

`brew install go`

Then Hugo of course: 

`brew install hugo`

I suggest then to create a directory, called websites, that you will use to host your website, and maybe others:

`mkdir websites` and then `cd websites`. 

Run now the Hugo command to create your static website:

`hugo new site yourwebsitename`

You should see the following propmpt:

```
Congratulations! Your new Hugo site is created in /Users/YourName/Sites/yoursitename.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
Choose a theme from https://themes.gohugo.io/, or
create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.

```

Comgratulations! you hugo site is created, now time to make it beautiful and add some content. 

Hugo provides a list of [themes](https://themes.gohugo.io/) for that. The theme can be easily downloaded and put in the ‘/themes‘ folder, but i really recommend to install it as a ‘git‘ ‘submoduile‘ in such a way to make it easily updatable:

```
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
echo 'theme = "ananke"' >> config.toml
```

You can then put some content, like creating your first blog post:

`hugo new posts/website-first-post.md`

That's it, we can now run the local Hugo server and see if everything is working fine.

`hugo server -D` 

Connect now to `localhost:1313` and check if your website is up and running. 

Now it's time to publish our newly created static website. We are gonna used as i mentioned earlier Github Pages. 
Github Pages is a free static hosting service offered by Github meant to be used for personal or project pages. Deploying a website connected to the url `<githubusername>.github.io` is quite straighforward. 
All it takes is:

```
mkdir <username>.github.io` && `cd <username>.github.io
git init
git remote add origin <github ssh>
touch index.html
echo "<html> hello world<!html>" >> index.html
git add . 
git commit -m "Hello world."
git push
```

Now that we saw how to do it we can do the same for our Hugo. 

First let's generate the static content, we will choose to set the content on the folder `<githubusername>.github.io`. 
We can do this by setting the path on the file `config.toml`: `publishdir = "<your-website-directory>" specifying in this way the folder the generator is supposed to write the static content into. 

After that, let's generate the content:

`hugo`

Once the command finished you can notice on the folder `<githubusername>.github.io` that a lot of files have been added. 
It's time to commit our hard work and enjoy our new website! 
```
git add . 
git commit -m "Hugo website."
git push
```

You can now connect to `<githubusername>.github.io` and check that everything works properly. 

If you want you can add a custom url, for that you can purchase it on any domain register sites, i used [dynadot](https://dynadot.com) for this.
After you must connect your domain to the website itself. You can do that by creating an `A Record` on the domain service of your choice, although usually domain register offer one to use directly. The `A Record` must point the domain to this list of IP addresses: `185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153`. 

You also need to add a file `CNAME`, to your github repository, the file must contain a single line with the domain you registered. 

That's it! Thanks for reading this post. 
