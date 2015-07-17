# Dress to Impress development environment

Oh, awesome! You wanna contribute to Dress to Impress? I'm super excited to have someone working on this who's not just me! xD

## Installation

### Installing Git

First off: if you don't have the Git client on your computer yet, that's important for working with Github. [Installing git is easy][git-install], especially on Ubuntu:

    sudo apt-get install git

  [git-install]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git


### Installing Vagrant + Ansible

To keep the environment consistent, automatically reproducible, and far away from the other stuff on your machine, we develop Dress to Impress inside a virtual machine. [Vagrant][vagrant] and [Ansible][ansible] help us create and manage this VM so that the process is transparent to you :)

1. [Install the latest version of Vagrant.][vagrant-install] This will help you create the VM itself.
2. [Install the development version of Ansible.][ansible-install] This will help you install Dress to Impress onto the VM.
    * We use features from Ansible 2.0 (like the [Bundler module](http://docs.ansible.com/bundler_module.html)), but 2.0 was in development at time of writing, so we used the development branch.
    * If a stable build of 2.x is available now, try that instead.

For example, on Ubuntu, the install process looks something like this:

    sudo apt-get install vagrant
    cd ~
    git clone git://github.com/ansible/ansible.git --recursive
    echo -e '\nsource ~/ansible/hacking/env-setup > /dev/null' >> ~/.bashrc
    bash

  [vagrant]: https://www.vagrantup.com/
  [vagrant-install]: https://www.vagrantup.com/downloads.html
  [ansible]: http://docs.ansible.com/index.html
  [ansible-install]: http://docs.ansible.com/intro_installation.html#running-from-source


### Installing the Dress to Impress development environment

Sweet. Now we're ready to do DTI-specific stuff :D

First, clone this repository to your local machine, anywhere you like.

    git clone https://github.com/openneo/impress-dev-environment.git
    cd impress-dev-environment

Next, install the Ansible modules that we depend on:

    sudo ansible-galaxy install -r requirements.txt

Now we're ready to create a Dress to Impress VM. This might take a few minutes.

    vagrant up

If all goes well, then, hooray! We're done! Now you can visit [impress.dev.openneo.net](http://impress.dev.openneo.net/) in your web browser, and it'll forward the request to the local version of Dress to Impress running in the VM.

## Development

Lots more to talk about in here. The gist is that `apps/impress` is the main app, and changes in there will automatically propagate to the live copy of the app. `apps/neopia` is the app that runs the pet loading API, and loading new code is a bit trickier: for now, you'll wanna `sudo -i`, then `go install github.com/openneo/neopia`, then `service neopia restart`, then `exit` to get out of scary sudo mode. The details of all this and running all the various components will probably get expanded out into wiki pages if there's sufficient interest in contributing :)

The dev environment is currently missing a few key components:

* Search
* Authentication
* Image processing

But you *can* load pets and wear/unwear stuff, so that's exciting!

### Expected Hiccups

You'll probably note some ugly behavior that also appears on the live copy of DTI: the pet loading server forwards the user to the wardrobe page *while* it's notifying DTI of the new data. So, if you enter the name of a pet we've never seen before, the wardrobe might bump you back out because it's a bad species/color combination. Or, in lesser cases, you might just see No Data Yet on an item, even though you just modeled it.

This is a handy optimization to keep the pet server lean, but it leads to ugly behavior for contributors; I wonder if maybe we can have Neopia block for a second or two, waiting for a DTI response, then give up if it takes too long? Yeah, that's a valid feature if someone wants to learn Go and add it xP
