---
title: 'Create Your Free Developer Blog Using Hugo'
resources:
  - name: 'featured-image'
    src: 'featured-image-en.png'
categories: ['documentation']
tags: ['hugo', 'installation']
date: 2021-03-22T09:31:44+01:00
draft: false
---

## The Role of Hugo

After doing some research I decided to use **Hugo**. Hugo is an open-source static site generator. Static site generators build a web page once, at the moment you’re creating new content or editing it.

Static web page give you **100%** control over your content and web design. Because Hugo is all about static site, it lacks fewer security issues. There’s nothing to be exploited on the server-side. No PHP running. Nothing.

This makes static site quite stable against security breaches.

## Using Hugo

As mentioned, Hugo is open source and can be installed quite easily. If you’re using Mac (like me), you can use Homebrew to install Hugo:

### Homebrew & Git

If you don't have Homebrew installed, then you can install it with:

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Make sure everything is up to date & install Git

```shell
brew update
brew install git
```

### Hugo

#### Install

Install Hugo with Homebrew

```shell
brew install hugo
```

Once the installation is complete, you can issue the command:

```shell
hugo version
```

To ensure the installation was successful you need to see something like:

{{< admonition type=info open=true >}}
hugo v0.81.0+extended darwin/amd64 BuildDate=unknown
{{< /admonition >}}

#### Generate the Site

{{< admonition type=note open=true >}}
Please, replace **example.com** with your Domain or with Site Name
{{< /admonition >}}

```shell
hugo new site example.com
```

That should complete in the blink of an eye. You will now have a new directory for your site. Change into that directory with the command:

```shell
cd example.com
```

#### Directory structure

Running the `hugo new site` generator from the command line will create a directory structure with the following elements:

```
.
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

{{< admonition type=note open=true >}}
If you want to know more about Directory Structure you can learn it on officially pages {{< link href="https://gohugo.io/getting-started/directory-structure/" content=GoHugo >}}.
{{< /admonition >}}

For us is the most important file `config.toml` where we need to specify all of our settings.

#### Using Theme

Because we want to spend all our free time with our families and we already working more than eight hours per day is the use of a theme the best choice to start. Personally, I spent few hours finding the best blog theme for me. I ended with {{< link href="https://github.com/sunt-programator/CodeIT/" content=CodeIT >}} theme.

Firstly I tried to use {{< link href="https://github.com/dillonzq/LoveIt" content=LoveIT >}} theme, but this theme is not maintained anymore and I found that **CodeIT** is. There is a lot of themes almost the same with some changes, but I like this one the most. So how did I use it to serve my needs?

Because I wanted to have my code hosted in Github, and so did you, we need to initialize a git repository first.

```shell
git init
```

Make **CodeIT** repository a submodule of our directory:

```shell
git submodule add https://github.com/sunt-programator/CodeIT.git themes/CodeIT
```

The **git submodule add** command will add folder **CodeIT** to `/themes` and create file `.gitmodules` with:

```
[submodule "themes/CodeIT"]
path = themes/CodeIT
url = https://github.com/sunt-programator/CodeIT.git
```

Now we have the theme loaded and We just need to edit `.config.toml` to use our theme.

Here is the basic `.config.toml` file to test our theme:

```toml
baseURL = "http://example.com/"
defaultContentLanguage = "en"
languageCode = "en"
title = "My New Hugo Site"
theme = "CodeIT"

[params]
  version = "0.2.X"

[menu]
  [[menu.main]]
    identifier = "posts"
    pre = ""
    post = ""
    name = "Posts"
    url = "/posts/"
    title = ""
    weight = 1
  [[menu.main]]
    identifier = "tags"
    pre = ""
    post = ""
    name = "Tags"
    url = "/tags/"
    title = ""
    weight = 2
  [[menu.main]]
    identifier = "categories"
    pre = ""
    post = ""
    name = "Categories"
    url = "/categories/"
    title = ""
    weight = 3

[markup]
  [markup.highlight]
    noClasses = false
```

{{< admonition type=note open=true >}}
If you want to edit it to your needs, here is the link to official {{< link href="https://codeit.suntprogramator.dev/theme-documentation-basics/" content=documentation >}} or you can get inspiration from my {{< link href="https://github.com/jozefrebjak/jozefrebjak.com/blob/master/config.toml" content=config >}}.
{{< /admonition >}}

#### Launching the Website Locally

{{< admonition type=note open=true >}}
Since the theme use `.Scratch` in Hugo to implement some features, it is highly recommended that you add `--disableFastRender` parameter to hugo server command for the live preview of the page you are editing.
{{< /admonition >}}

Launch by using the following command:

```shell
hugo serve --disableFastRender
```

If is everything fine, you will see:

```
Built in 105 ms
Watching for changes in /Users/USER/example.com/{archetypes,content,data,layouts,static,themes}
Watching for config changes in /Users/USER/example.com/config.toml
Environment: "development"
Serving pages from memory
Web Server is available at localhost:1313
Press Ctrl+C to stop
```

Go to {{< link "localhost:1313" >}} to see your running site.

{{< figure src="new-hugo-site.png" >}}

#### Add first post

You can create new content with a command like:

```sh
hugo new posts/blog-post-1.md
```

Command will create file `blog-post-1.md` in `/content/posts` folder.

Edit the content of the new file as you need. Save and close the file and Hugo will automatically detect the change of the newly added blog post:

For example:

```md
---
title: 'Blog Post 1'
date: 2021-03-22T12:11:52+01:00
draft: false
---

# Test Header

Test text
```

{{< figure src="new-hugo-site-post.png" >}}

Once you have the site exactly how you want it, kill the daemon server with the [Ctrl]+ keyboard combination and build the site with the command (run from the root directory):

```sh
hugo
```

The site will very quickly build and create a new public folder inside the document root. Upload that folder to your hosting server and you’re good to go.

## Conclusion

And that’s the creating a static website with Hugo. If you managed to make it all the way through this post then congratulations! In the next blog post, I will cover how to get free hosting of your blog on Netlify.

Netlify provides continuous deployment services, global CDN, ultra-fast DNS, atomic deploys, instant cache invalidation, one-click SSL, a browser-based interface, a CLI, and many other features for managing your Hugo website.
