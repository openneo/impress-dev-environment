# Dress to Impress development environment

Oh, awesome! You wanna contribute to Dress to Impress? I'm super excited to have someone working on this who's not just me! xD

## Installation

### Installing Git

First off: if you don't have the Git client on your computer yet, that's important for working with Github. [Installing git is easy][git-install], especially on Ubuntu:

    sudo apt-get install git

  [git-install]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git


### Installing Virtualbox + Vagrant + Ansible

To keep the environment consistent, automatically reproducible, and far away from the other stuff on your machine, we develop Dress to Impress inside a virtual machine. [Vagrant][vagrant] and [Ansible][ansible] help us create and manage this VM so that the process is transparent to you :)

1. [Install the latest version of Virtualbox.][virtualbox-install] This will help you run the VM.
    * Known to work with Virtualbox 4.3.28.
2. [Install the latest version of Vagrant.][vagrant-install] This will help you create the VM itself.
    * Known to work with Vagrant 1.7.3.
3. [Install the development version of Ansible.][ansible-install] This will help you install Dress to Impress onto the VM.
    * We use features from Ansible 2.0 (like the [Bundler module](http://docs.ansible.com/bundler_module.html)), but 2.0 was in development at time of writing, so we used the development branch.
    * Known to work with commit 37e38924de3473da32a85e7145745118bf83247a.
    * If a stable build of 2.x is available now, try that instead.
    * When building from source, don't forget to install the Jinja2 dependency.

For example, on 64-bit Ubuntu, the install process looks something like this:

    URL='https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.3_x86_64.deb'; FILE=`mktemp`; wget "$URL" -qO $FILE && sudo dpkg -i $FILE; rm $FILE
    sudo apt-get install virtualbox python-jinja2
    cd ~
    git clone git://github.com/ansible/ansible.git --recursive
    echo -e '\nsource ~/ansible/hacking/env-setup -q' >> ~/.bashrc
    bash

  [virtualbox-install]: https://www.virtualbox.org/wiki/Linux_Downloads
  [vagrant]: https://www.vagrantup.com/
  [vagrant-install]: https://www.vagrantup.com/downloads.html
  [ansible]: http://docs.ansible.com/index.html
  [ansible-install]: http://docs.ansible.com/intro_installation.html#running-from-source


### Installing the Dress to Impress development environment

Sweet. Now we're ready to do DTI-specific stuff :D

First, clone this repository to your local machine, anywhere you like.

    git clone https://github.com/openneo/impress-dev-environment.git
    cd impress-dev-environment

Next, download the Vagrant box and plugins we'll use. This might take a few minutes.

    vagrant box add ubuntu/trusty64
    vagrant plugin install vagrant-hostmanager

Next, install the Ansible modules that we depend on:

    ansible-galaxy install -r requirements.txt

Now we're ready to create a Dress to Impress VM. This might take a few minutes.

    vagrant up

If all goes well, then, hooray! We're done! Now you can visit [impress.dev.openneo.net](http://impress.dev.openneo.net/) in your web browser, and it'll forward the request to the local version of Dress to Impress running in the VM.

## Development

Lots more to talk about in here. The gist is that `apps/impress` is the main app, and changes in there will automatically propagate to the live copy of the app. `apps/neopia` is the app that runs the pet loading API, and loading new code is a bit trickier: for now, you'll wanna `sudo -i`, then `go install github.com/openneo/neopia`, then `service neopia restart`, then `exit` to get out of scary sudo mode. The details of all this and running all the various components will probably get expanded out into wiki pages if there's sufficient interest in contributing :)

We clone the public `openneo/impress` repository into `apps/impress` by default. If you want to develop against your own fork, delete the `apps/impress` directory and clone your fork there in its place.


### Known Issues


#### Item page doesn't show the species thumbnails

On live DTI, when you visit the page for a particular item, you can try it out on each species. Your dev DTI, though, doesn't know the URL for a Blue Acara thumbnail, so it can't show the images. We can't *really* gather this automatically because it requires logging in, but I've grabbed them before for live DTI, and we can copy them over here; the real question is probably where the data should go. (Probably in `db/seeds.rb`?)


#### Modeling a new pet type sends you back to the homepage

This happens on the live site, too: when you model a new pet/color combo in the big box on the homepage, you get redirected to the wardrobe page, then redirected back. That's because the pet loading server redirects you to DTI *while* it's sending the pet data to DTI, so you might get there before the data does.

This is a really handy optimization for most cases, since it *only* matters when the user is going to the wardrobe (which we could detect in advance) and they've contributed new data (which we can't detect in advance). For folks who are just modeling willy-nilly or who are using their own pets as prototypes, this speeds up their experience and keeps the pet server non-blocking. But, for contributors, it probably makes for a bit of an ugly experience.

There are two major approaches we could take here:

* We could have the pet server wait for a little bit on the DTI data submission. It should probably give up pretty quick, but this could help mitigate most of the failed races.
  - But note, too, that the pet data handler delegates some of the work to background tasks. I think it's just the search indexing, though, so that's not part of the race anyway…
* Or, we could have the wardrobe frontend realize that the user says that the Neopia server says that this data exists, even though the wardrobe backend doesn't have it yet, and display some kind of message.
  - We could have the wardrobe page spin for a bit a retry until the data loads.
  - Or, we could have the wardrobe page bounce the user back a bit more politely than it does now. Since this isn't a big use case, and it mostly affects our power users, I think I'm okay with the bouncing so long as we're nice about it.
    + Heck, you don't even *need* to detect *why* the resource was missing; you could just bounce back on any failed resource and say "sometimes new data takes a while blah blah". But detecting would be nicer.
  - Note, too, that the current bounce-back behavior is actually a coincidence of how we implement picking a bad species/color combo; we hit Back on the history, which usually bounces them back to their last wardrobe state rather than out of the app entirely. If we're doing this for *all* newly-modeled data, we'll need to think about how to detect and handle failed item and asset loads, too—especially since they're the even more common case.


#### There's zero image processing

SWFs aren't converted into PNGs, and those PNGs aren't layered on top of each other to produce outfit thumbnails. I think this one's mostly a matter of installing Gnash and ImageMagick properly; heck, it might even be as easy as adding a few apt packages. But it's noncritical, and I don't feel like poking around in there yet ;P If you feel called to it, though, go for it! (We'll also learn whether FakeS3 actually works as expected xP)
