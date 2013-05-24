# Cookbooks

A set of cookbooks for various Vagrant-based development environments.

## Setup

I keep all my code in `~/Projects`.

To setup this set of cookbooks go to `~/Projects` or equivalent and clone:

    git clone git://github.com/filiptepper/cookbooks.git

Here's an example annotated `Vagrantfile`:

    Vagrant.configure('2') do |config|

      # I have tested all bundled recipes with Ubuntu 13.04 image.

      config.vm.box     = 'raring'
      config.vm.box_url = 'http://cloud-images.ubuntu.com/vagrant/raring/current/raring-server-cloudimg-amd64-vagrant-disk1.box'

      # For Rails projects I use default web server at 3000.

      config.vm.network :forwarded_port, :guest => 3000, :host => 3000

      # I use NFS to improve performance.
      # For more go to http://docs.vagrantup.com/v2/synced-folders/nfs.html

      config.vm.synced_folder '.', '/vagrant', :nfs => true
      config.vm.network :private_network, :ip => '192.168.243.243'

      # VirtualBox setup

      config.vm.provider :virtualbox do |vb|

        # Use 1GB of memory

        vb.customize [
          'modifyvm', :id,
          '--memory', '1024'
        ]

        # Use 2 CPU units

        vb.customize [
          'modifyvm', :id,
          '--cpus', 2
        ]
      end

      config.vm.provision :chef_solo do |chef|

        # Relative path to cookbooks bundle

        chef.cookbooks_path = '../cookbooks'

        # chef-solo setup

        chef.add_recipe 'build-essential'
        chef.add_recipe 'ruby_build'
        chef.add_recipe 'rbenv::system'
        chef.add_recipe 'rbenv::vagrant'
        chef.add_recipe 'postgresql::client'
        chef.add_recipe 'postgresql::server'
        chef.add_recipe 'imagemagick'

        chef.json = {
          :postgresql => {
            :password => {
              :postgres => 'postgres'
            }
          },
          :rbenv => {
            :rubies => [
              '2.0.0-p195'
            ],
            :gems => {
              '2.0.0-p195' => [
            ],
            :gems => {
              '2.0.0-p195' => [
                { :name => 'bundler' }
              ]
            }
          }
        }
      end
    end

Once you boot instance with `vagrant up` SSH to instance with `vagrant ssh`
and use rbenv to select Ruby version:

    cd /vagrant
    rbenv local 2.0.0-p195

Now you're free to `bundle install` and get things done!