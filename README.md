# Using Vagrant
### Choose a Box
The first thing you’ll need to do is pick a box that you want to build from. The `Vagrant Cloud` lets you easily find boxes that people have shared.

### Initialize and Start the Vagrant Box
After you’ve chosen a box, initialize the Vagrant box.
```shell
vagrant init hashicorp/precise64
```

Create a new folder and cd into that. The following command will create a Vagrant file for you. 
```shell
vagrant up
```

### SSH into the Box and Customize It
We’ll now SSH into the box and start customizing it.
```shell
vagrant ssh
```

Now, we need to setup our server by installing whatever we want on it.
```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install vim
sudo apt-get install apache2
sudo service apache2 restart
```
### Make the Box as Small as possible
We’re now going to clean up disk space on the VM so when we package it into a new Vagrant box, it’s as clean as possible. First, remove APT cache
```shell
sudo apt-get clean
```
Then, “zero out” the drive (this is for Ubuntu):

```shell
sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
```

Lastly, let’s clear the Bash History and exit the VM:
```shell
cat /dev/null > ~/.bash_history && history -c && exit
```

### Repackage the VM into a New Vagrant Box
We’re now going to repackage the server we just created into a new Vagrant Base Box. It’s very easy with Vagrant:
```shell
vagrant package --output mynew.box
```
### Add the Box into Your Vagrant Install
The previous command creates a “mynew.box” file. You can technically put this wherever you want on your computer. Now, let’s add this new Vagrant Box into Vagrant:
```shell
vagrant box add mynewbox mynew.box
```

This now will add the box into your Vagrant install allowing to initiate this from any folder, but before we do this, let’s delete and remove the Vagrant file we built this box from.
```shell
vagrant destroy
rm Vagrantfile
```
### Initialize Your New Vagrant Box
We need to now initialize a Vagrant environment from our brand new box using the same command from earlier but referencing the new Box.
```shell
vagrant init mynewbox
```
## Customize Your New Vagrantfile
When you initialize the Vagrant environment, it creates a Vagrantfile for you. Open the Vagrantfile and delete everything. Now paste the following bare-bones code into your Vagrantfile:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "mynewbox"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = "mynewbox"
  config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=666"]

end
```

## SHELL PROVISIONER
The shell provisioner allows you to upload and execute a script within the guest machine. The shell provisioner takes various options.
`inline (string)` - Specifies a shell command inline to execute on the remote machine. See the inline scripts section below for more information.
`path (string)` - Path to a shell script to upload and execute. It can be a script relative to the project Vagrantfile or a remote script (like a gist).

### INLINE SCRIPTS
Perhaps the easiest way to get started is with an inline script. An inline script is a script that is given to Vagrant directly within the Vagrantfile. An example is best:
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell",
    inline: "echo Hello, World"
end
```

This causes echo `Hello, World` to be run within the guest machine when provisioners are run.

Combined with a little bit more Ruby, this makes it very easy to embed your shell scripts directly within your Vagrantfile. Another example below:

```ruby
$script = <<SCRIPT
echo I am provisioning...
date > /etc/vagrant_provisioned_at
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: $script
end
```

The script is assigned to a global variable `$script`. This global variable contains a string which is then passed in as the inline script to the Vagrant configuration.

### EXTERNAL SCRIPT
The shell provisioner can also take an option specifying a path to a shell script on the host machine. Vagrant will then upload this script into the guest and execute it. An example:
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "script.sh"
end
```
Relative paths, such as above, are expanded relative to the location of the root Vagrantfile for your project. Absolute paths can also be used, as well as shortcuts such as ~ (home directory) and .. (parent directory).

If you use a remote script as part of your provisioning process, you can pass in its URL as the path argument as well:

```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "https://example.com/provisioner.sh"
end
```

### SCRIPT ARGUMENTS
You can parameterize your scripts as well like any normal shell script. These arguments can be specified to the shell provisioner. They should be specified as a string as they'd be typed on the command line, so be sure to properly escape anything:
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell" do |s|
    s.inline = "echo $1"
    s.args   = "'hello, world!'"
  end
end
```

You can also specify arguments an array if you don't want to worry about quoting:
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell" do |s|
    s.inline = "echo $1"
    s.args   = ["hello, world!"]
  end
end
```