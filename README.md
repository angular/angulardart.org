# Angulardart.org

Your changes to the master branch of angular/angulardart.org should become
visible on angulardart.org in 5-10 minutes.

If you're making a change that isn't simple, please build the site locally
before you commit the changes. 
Here's how:

## Automatic deploy (via Vagrant)

1. Download [Vagrant](https://www.vagrantup.com/downloads.html).
2. Open root folder of cloned repository.
3. Run `vagrant up`. On subsequent runs execute `vagrant provision` after `vagrant up`.
4. View the site at `10.0.0.2:4000` (10.0.0.2 address is from `Vagrantfile`, you can change it).

## Manual deploy

1. If `which bundle` returns something, you have the prerequisites. Otherwise:
  1. Download rubygems from http://rubygems.org/pages/download
  2. Install rubygems with `ruby setup.rb`
  3. Install bundler with `gem install bundler`

2. Run `bundle install` from within the directory that contains the `Gemfile`
  * If you have issues installing posix-spawn
    (messages like "posix-spawn.c:6:19: error: errno.h: No such file or directory")
    do the following and then run `bundle install` again:

    `xcode-select --install`

3. Run `cd site`.

4. Run `jekyll serve --watch`. This runs the dev server.
   If you get a horrible set of messages, **make sure you're in site/.**

5. View the site at `localhost:4000`.
