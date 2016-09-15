---
layout: post
title: Using GitHub Pages with a dockerized Jekyll
---

Whenever I try some new piece of software, I try to keep all the requirements
as close to each other  as possible using tools like chruby, nvm, virtualenv,
etc.  Docker is a great way of achieving this. Its not just for containing
individual service deployments, but it can contain a single tool just as well.

## Set up

Jekyll publishes a Docker container which:

> ...provides an isolated Jekyll instance with the latest version of
> Jekyll and a bunch of nice stuff to make your life easier when working
> with Jekyll...

Let's put it to work and create a place for your GitHub Pages:

```bash
mkdir -p ~/ws/pages
cd ~/ws/pages

docker pull jekyll/jekyll
docker run --rm --label=jekyll \
  --volume=$HOME/ws/pages:/srv/jekyll \
  -it -p 127.0.0.1:4000:4000 jekyll/jekyll \
  jekyll new .

git init
git add --all
git commit --message "jekyll new'
```

Configure Jekyll by editing the ``_config.yml`` file. Make sure to set ``url:
https://$USER.github.io``. By default your GitHub Pages site will be HTTPS
only. If your jekyll config is set to HTTP, your site will not be able to find
its own files and probably appear without any styling.

Jekyll can also serve your pages locally.

```bash
docker run --rm --label=jekyll --volume=$PWD:/srv/jekyll \
  -it -p 127.0.0.1:4000:4000 jekyll/jekyll jekyll serve --drafts
```

Keep this command open in a separate terminal, it will watch your Jekyll
directory for changes and keep the local site up-to-date. You can browse your
site at [localhost:4000](http://localhost:4000/) 

You should now have a working Jekyll site which we'll test by writing a
draft article:

```bash
mkdir -p $HOME/ws/pages/_drafts
cat > $HOME/ws/pages/_drafts/my-first-post.md << EOF
---
layout: post
---
# Work in Progress
This will be a great post!
EOF
```

Draft articles will appear at the top of your articles. You should now
see some activity in the Jekyll terminal and your site should contain the
draft article when you refresh.

## Publishing to Github

There are two ways of publishing your site. The easiest is to let GitHub do all
the work and simply push your Jekyll site directly to your
``username.github.io`` repository. GitHub will serve the generated site at
``https://username.github.io``.

First, let's finish our article:

```bash
echo "And now it is..." >> $HOME/ws/pages/_drafts/my-first-post.md
mv _drafts/my-first-post.md _posts/$(date +%Y-%m-%d)-my-first-post.md
```

If you commit your changes and push to the ``master`` branch of your repository
then your post should now be visible at GitHub.

```bash
git commit -a -m 'finished my first post'
git remote add origin git@github.com:$USER/$USER.github.io
git push -u origin master
```

### Publishing a generated site to GitHub

GitHub does not support all plugins and/or markdown dialects. Because of this
you might want to publish a static site to github.

Create a github repository called ``username.github.io`` and clone it into the
``_github`` directory. The github checkout is intentionally kept in a separate
directory (i.e. not ``_site``) because at some point I will accidentally
publish my drafts to github...

```bash
cd $HOME/ws/pages
git clone git@github.com:$USER/$USER.github.io _github
echo _github >> .gitignore

docker run --rm --label=jekyll \
  --volume=$PWD:/srv/jekyll \
  -it -p 127.0.0.1:4000:4000 jekyll/jekyll \
  jekyll build --destination _github

cd _github
git add --all
git commit -m 'published my first post'
git push
```

Your post should now be visible at GitHub.

## Google Analytics and Comments

By default Jekyll uses the [Minima theme](https://github.com/jekyll/minima) and
it supports precisely the two things I wanted on my blog: google analytics and
comments. All you need to do is register your site at those two sites and add a
config entry for each in your ``_config.yml``.
