VOLUME_REMOVE = "YES"

Vagrant.configure('2') do |config|

  config.vm.synced_folder ".", "/srv/data", type: "rsync", rsync__exclude: [".git/", "vars.yml"]

  config.vm.define "mydroplet" do |config|
      config.vm.provider :digital_ocean do |provider, override|
        override.ssh.private_key_path = '~/.ssh/docean'
        override.vm.box = 'digital_ocean'
        override.vm.box_url = "https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box"
        override.nfs.functional = false
        override.vm.allowed_synced_folder_types = :rsync
        provider.token = ENV['DOKEY']
        provider.image = 'ubuntu-18-04-x64'
        provider.region = 'fra1'
        provider.size = 's-1vcpu-1gb'
        provider.ssh_key_name = 'mykey'
        provider.backups_enabled = false
        provider.private_networking = false
        provider.ipv6 = false
        provider.monitoring = false
        provider.tags = ['tag1', 'tag2']
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
