# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

VAGRANT_API_VERSION = "2"

alt_id = Time.new.strftime("%Y%m%d%H%M%S")

COMMIT_ID        = ENV["cmid"] || alt_id
VAGRANT_HOSTNAME = "devhost-#{COMMIT_ID}"
VAGRANT_BOX_NAME = "AntonioMeireles/coreos-stable"
MOUNT_DIR        = "/vagrant/docker"
ROLES_DIR        = "/etc/ansible/roles/role_under_test"
VAGRANT_IP       = "172.16.78.100"
TESTS_CONTAINERS = "tests/tests_containers.yml"

$vm_memory = 2048
$vm_cpus = 2
$vb_cpuexecutioncap = 100

unless File.file?(TESTS_CONTAINERS)
  puts "Containers File: #{TESTS_CONTAINERS} Absent"
  exit 1
end

containers = YAML.load_file(TESTS_CONTAINERS)

Vagrant.configure(VAGRANT_API_VERSION) do |config|

  config.ssh.insert_key = false

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.box = "#{VAGRANT_BOX_NAME}"
  config.vm.network "private_network", ip: "#{VAGRANT_IP}"

  $shared_folders = [['./', "#{MOUNT_DIR}"]]
  $shared_folders.each_with_index do |(host_folder, guest_folder), index|
    config.vm.synced_folder host_folder.to_s, guest_folder.to_s,
      id: "core-share%02d" % index,
      nfs: true,
      mount_options: ['nolock,vers=3,udp']
  end

  config.vm.provider "virtualbox" do |v|
    v.name                  = "#{VAGRANT_HOSTNAME}"
    v.check_guest_additions = false
    v.functional_vboxsf     = false
    v.gui                   = false
    v.memory                = $vm_memory
    v.cpus                  = $vm_cpus
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "#{$vb_cpuexecutioncap}"]
  end

  containers.each do |container|
    container_name     = container["name"] + "-" + "#{COMMIT_ID}"
    container_init     = container["init"]
    container_opts     = container["opts"]
    container_image    = container["image"]

    config.vm.provision "docker-run", type: "docker", run: "always" do |d|

      d.run "#{container_name} ", image: "#{container_image}",
        cmd: "ansible-playbook "\
             "-i #{ROLES_DIR}/tests/inventory "\
             "-vv #{ROLES_DIR}/tests/test_local.yml "\
             "-e 'vnri_role_setup=True' "\
             "-c local "\
             "-s "\
             "-t vrni_setup",
        args: "#{container_opts} "\
              "-v #{MOUNT_DIR}:#{ROLES_DIR}:rw "\
              "#{container_image} "\
              "#{container_init}"

    end
  end

end
