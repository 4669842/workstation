### Shortlist

* git
* mercurial
* [https://github.com/CERN-CERT/WAD](https://github.com/CERN-CERT/WAD)
* TODO: some commandline linkchecker
  * https://pypi.python.org/pypi/existence/0.2.6
    * [ ] patch to work with URLs

### Goals

* gather scenarios on how to quickly prepare (development)
environment for different tasks

* improve user experience for such scenarios

* collect list of those "different tasks"

### Workstation

..is a box (deck) with some operating system and personal
tools to do the job.

The basic stuff I expect to see on every Linux machine:

  (browser: all firefox, google-chrome)
  git
  hg
  vim

Windows is the same, except that vim is replaced with
Far Manager + Colorer.

Below are these requirements adapted for the clean
installs of some operating systems.

== Workstation: Linux, Fedora 17

  git
  hg
  vim

  google-chrome


= Bootstraping workstations
== ..with Chef Solo on Linux

Too long/didn't read:

  git clone git://github.com/techtonik/workstation.git
  cd workstation/chef
  sudo yum install rubygems ruby-devel gcc
  sudo gem install chef --no-ri --no-rdoc
  sudo gem install librarian --no-ri --no-rdoc
  librarian-chef install
  sudo `which chef-solo` -c config-solo.rb
  
Now in more detail.

Chef Solo is a standalone tool of Chef framework
(framework for infrastructure automation). Meant as
a simple alternative to full Chef client/server and
workstation stack, it is crippled. For example, knife
- tool to manage cookbooks and more - doesn't work
with it.

Chef Solo docs are at:
http://wiki.opscode.com/display/chef/Chef+Solo

The power of Chef is in reusable (and usually cross-
platform) Cookbooks that are already available. For
example, you probably don't know exact package names
for installing 'vim' on different systems. This is
already written in external cookbook. You can manually
download it, but it suxx. Probably with Chef server it
is easier, but because Chef Solo is crippled, you need a
tool to manage these external cookbooks - a good tool is
Librarian-Chef.

=== 1. install Chef (solo is included) and Librarian

(tested: Fedora 17, Chef 10.12.0):
  sudo yum install rubygems ruby-devel gcc
  sudo gem install chef --no-ri --no-rdoc

Install Librarian for managing external cookbooks:
  sudo gem install librarian

=== 2. bootstrap - download cookbooks, run chef

Chef-Solo is crippled, so it can not download dependent
cookbooks. This is made by Librarian-Chef, who reads its
own configuration Cheffile.

First you need to create/get Chef Repository, where you
create Cheffile. Mine is here:

  git clone git://github.com/techtonik/workstation.git
  cd workstation/chef

Fetch dependencies:

  librarian-chef install

Run chef-solo for local repository:

  sudo `which chef-solo` -c config-solo.rb

`which` is required here, because by default chef-solo is
not in PATH for `root` user on Fedora. 

Notes: When started I hoped that I could just install
chef-solo and run it against remote GitHub URL so that it
could get recipe, figure out dependent cookbooks, resolve
dependencies, download them and run. In Chef 10.12.0 this
scenario is not ready (yet?).

=== 3. enjoy cooking solo


