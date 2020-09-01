# Vagrant + Ansible DigitalOcean example

This example shows how to call Ansible from Vagrant and create/attach DO Block Storages. 

## Usage

### Install Vagrant DO plugin

```bash
vagrant plugin install vagrant-digitalocean
```

### Configure vars.yml

```yml
---
# your DigitalOcean API Token:
dokey: "TOKEN"

# Volumes count (must be equal to <mount_points> list length)
block_count: 1
# Volumes size in GB
block_size: 50
# Region
block_region: fra1
# Volume name (will be supplemented by index of the block storage)
volume_name: myvolume
# State: present for create or absent for remove
volume_state: present

# List with mount points
mount_points: ["/srv/data"]
# State (see docs: https://docs.ansible.com/ansible/latest/modules/mount_module.html#parameter-state)
mount_state: mounted
# Filesystem type
fstype: ext4
```

### Configure Vagrantfile

Set environment variable `DOKEY` for Vagrant:

```bash
export DOKEY=<YOUR_DO_TOKEN>
```

```ruby
# If "YES" - volumes (Block Storages) will be also removed after droplet destroying
VOLUME_REMOVE = "YES"

Vagrant.configure('2') do |config|

  # Custom synced folder
  config.vm.synced_folder ".", "/srv/data", type: "rsync", rsync__exclude: [".git/", "vars.yml"]

  config.vm.define "mydroplet" do |config|
      config.vm.provider :digital_ocean do |provider, override|
        # specify path to your SSH private key file
        override.ssh.private_key_path = '~/.ssh/docean'
        override.vm.box = 'digital_ocean'
        override.vm.box_url = "https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box"
        override.nfs.functional = false
        override.vm.allowed_synced_folder_types = :rsync
        provider.token = ENV['DOKEY']
        # Choose image, region and 'size' of droplet as you want
        provider.image = 'ubuntu-18-04-x64'
        provider.region = 'fra1'
        provider.size = 's-1vcpu-1gb'
        # SSH key name in DO
        provider.ssh_key_name = 'mykey'
        provider.backups_enabled = false
        provider.private_networking = false
        provider.ipv6 = false
        provider.monitoring = false
        #provider.tags = ['tag1', 'tag2']
      end

      config.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
      end

      config.trigger.before :destroy do |trigger|
        if VOLUME_REMOVE == "YES"
            trigger.info = "Destroying volumes..."
            trigger.run = {inline: "ansible-playbook playbook.yml -e volume_state=absent"}
        else
            trigger.info = "Skip volumes destroying..."
        end
      end
  end
end
```

### Create a droplet

```bash
vagrant up --provider=digital_ocean
```

### Delete a droplet

```bash
vagrant destroy -f
```

## Links

For more information see:

[Vagrant DO Plugin](https://github.com/devopsgroup-io/vagrant-digitalocean)\
[Digital Ocean API](https://developers.digitalocean.com/documentation/v2/)\
[Ansible mount module](https://docs.ansible.com/ansible/latest/modules/mount_module.html)
