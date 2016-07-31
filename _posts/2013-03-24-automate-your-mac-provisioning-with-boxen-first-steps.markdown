---
layout: post
title: "Automate your Mac provisioning with Boxen (first steps)"
date: 2013-03-24 17:50
comments: true
---

One of the coolest tools that I have discovery recently is Boxen. For those who don't know it yet, a brief overview below. If you already know it, you can jump to the next section. The goal of this post is to share some basic knowledge about it to get you started.

## What is Boxen?

Boxen is a framework. It basically uses [Puppet](http://docs.puppetlabs.com/) to automate your Mac provisioning. In fact, it aims to standardize configs for a whole organization, which is how they use it at [Github](http://github.com). The benefit of it is pretty clear: imagine that a new Java vulnerability has been found. When using Boxen, TechOps people could push to the organization's repository a change that disables Java from the browser's preferences. Employees could just pull those changes, and Boxen would do the rest. Much better than sending an e-mail and expecting everyone to disable it, right?

The same would apply to many other situations, but you can get the idea. In fact, the driven behind Boxen was the long time that new hires spend configuring their workstations, until they can start working for real (AKA code).

I suggest looking at this presentation if you really want to dig into its philosophy: [BOXEN by Will Farrington](https://speakerdeck.com/wfarr/boxen)

## Setting up your boxen + environment

There are different ways to set up Boxen. As I use it just for my personal needs, I will focus on the simpler one.

The first thing needed is to install the **XCode Command Line Tools** (you don't need the full XCode). This is required just to have some basic tools, e.g., Git. This step is required if you using a brand new box - otherwise, as dev, you should have all the necessary tools already.

Having this dependency fulfilled, it's time to clone [our-boxen](https://github.com/boxen/our-boxen). That's just a template for you to start, which also includes some basic modules, such as `dnsmasq`, `gcc`, `git`, `homebrew`, `nginx`, etc. You can add/remove modules according to your needs - the next section will cover that.

Do the following to clone it:

{% highlight bash %}
sudo mkdir -p /opt/boxen
sudo chown ${USER}:admin /opt/boxen
git clone https://github.com/boxen/our-boxen /opt/boxen/repo
cd /opt/boxen/repo
git remote rm origin
git remote add origin <the location of my new git repository>
git push -u origin master
{% endhighlight %}

These steps can be found on the `our-boxen` link mentioned above.

We are basically cloning `our-boxen` to `/opt/boxen/repo`, and replacing its remote `origin` afterwards. This ensures that for now on your repository is the one who will keep track of the changes. That would be the same as forking it on Github to your account, and then cloning it to your box. However, by doing that the repo would become public, and as we are playing with the setup of your local box it's better that everything remains private - just in case you need to add some sensitive information for example.

Also note that it's important to keep this folder structure as boxen will later declare some environment variables pointing to `/opt/boxen`.

Now you can run **script/boxen**, and after that just append the following to your `~/.bashrc` or `~/.zshrc`:

{% highlight bash %}
[ -f /opt/boxen/env.sh ] && source /opt/boxen/env.sh
{% endhighlight %}

Open a new instance of your shell and run `boxen --env`.

If that runs smooth, then you are good to go :)

## Adding the Skype module and making it available

With your own boxen repo set up, it's time to fit it to your needs. There are many Puppet modules ready on the [boxen repo](https://github.com/boxen).

Boxen uses [librarian-puppet](https://github.com/rodjek/librarian-puppet) to manage Puppet modules. They even wrote a wrapper method to make it easier to fetch modules already available on the boxen repo.

Let's say you want to add Skype to your set up. In order to do that open a file called **Puppetfile**.

If you are not familiar with `librarian-puppet`, one easy way to understand it is that it does basically the same that `Bundler` does for gems. So `Puppetfile` plays the same role as a `Gemfile`.

Add Skype to the bottom of that file:

{% highlight puppet %}
# This file manages Puppet module dependencies.
#
# It works a lot like Bundler. We provide some core modules by
# default. This ensures at least the ability to construct a basic
# environment.

def github(name, version, options = nil)
  options ||= {}
  options[:repo] ||= "boxen/puppet-#{name}"
  mod name, version, :github_tarball => options[:repo]
end

# Includes many of our custom types and providers, as well as global
# config. Required.

github "boxen", "1.2.0"

# Core modules for a basic development environment. You can replace
# some/most of these if you want, but it's not recommended.

github "git",        "1.0.0"
github "dnsmasq",    "1.0.0"
github "homebrew",   "1.0.0"
github "ruby",       "1.0.0"
github "stdlib",     "3.0.0", :repo => "puppetlabs/puppetlabs-stdlib"
github "hub",        "1.0.0"

# Optional/custom modules. There are tons available at
# https://github.com/boxen.

github "skype",       "1.0.2"
{% endhighlight %}

Note that I added "1.0.2". That's (at the time of writing this post) the tag used in the repository to point to the latest version. If you go to [puppet-skype](https://github.com/boxen/puppet-skype), you can check all the tags available there (just click on button with the branch name and select the 'tags' tab).

Also note that we just typed "skype", and not the full repo's name, "puppet-skype". That's a convention that the boxen team created, which is defined in this wrapper method also available in the Puppetfile:

{% highlight ruby %}
def github(name, version, options = nil)
  options ||= {}
  options[:repo] ||= "boxen/puppet-#{name}"
  mod name, version, :github_tarball => options[:repo]
end
{% endhighlight %}

By looking at this piece of code you can also find out how to add a module which is not hosted on Github.

So let's say you want use your customized module of Skype instead, which is available on your private [BitBucket](https://bitbucket.org/) repo. Then, instead of:

{% highlight ruby %}
github "skype", "1.0.2"
{% endhighlight %}

You could say:

{% highlight ruby %}
mod "skype", :git => "git@bitbucket.org:yourusername/puppet-skype.git"
{% endhighlight %}

And that would be enough to make the Skype module available.

But we are still not using it. We must inform to Boxen (this is from Puppet in fact) that we want to include that module, so that it will actually install Skype when you run Boxen.

Again, I will cover the simplest way of doing that. Just open the file **manifests/site.pp** and include Skype's module on the bottom:

{% highlight ruby %}
...

node default {
  include dnsmasq
  include ruby
  include git
  include hub
  include homebrew

  include skype

...
{% endhighlight %}

After that, just run the **boxen** binary (it should be available as you ran `boxen --env` before) in your shell and wait. At the end of the proccess, Skype should be available for you. In fact, I suggest you to run ``boxen --debug``, which will give you a more verbose output of what's going on.

## What is next

I hope this covers the very basics of boxen. You can do all sort of cool things with it: clone your dotfiles, change an app's preferences and so on. I didn't want to complicate things too much though. When I started to play with it the docs were pretty bloated with unuseful information for starters, and that's the why I decided to write this post.

I plan to cover some more advanced aspects of Boxen soon, but this should be enough for anyone willing to start.

Any feedback is welcome :)

