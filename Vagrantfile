# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
GO_VERSION="1.6.2"

# Install Prereq Packages
sudo apt-get update
sudo apt-get install -y build-essential curl git-core vim unzip libseccomp-dev

# Setup go, for development of Nomad
SRCROOT="/opt/go"
SRCPATH="/opt/gopath"

# Get the ARCH
ARCH=`uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|'`

# Install Go
cd /tmp
wget -q https://storage.googleapis.com/golang/go${GO_VERSION}.linux-${ARCH}.tar.gz
tar -xf go${GO_VERSION}.linux-${ARCH}.tar.gz
sudo mv go $SRCROOT
sudo chmod 775 $SRCROOT
sudo chown vagrant:vagrant $SRCROOT

# Setup the GOPATH; even though the shared folder spec gives the working
# directory the right user/group, we need to set it properly on the
# parent path to allow subsequent "go get" commands to work.
sudo mkdir -p $SRCPATH
sudo chown -R vagrant:vagrant $SRCPATH 2>/dev/null || true
# ^^ silencing errors here because we expect this to fail for the shared folder

cat <<EOF >/tmp/gopath.sh
export GOPATH="$SRCPATH"
export GOROOT="$SRCROOT"
export PATH="$SRCROOT/bin:$SRCPATH/bin:\$PATH"
EOF
sudo mv /tmp/gopath.sh /etc/profile.d/gopath.sh
sudo chmod 0755 /etc/profile.d/gopath.sh
source /etc/profile.d/gopath.sh
SCRIPT

def configureVM(vmCfg)
  vmCfg.vm.box = "gbarbieru/xenial"

  vmCfg.vm.provision "shell", inline: $script, privileged: false
  vmCfg.vm.synced_folder '.', '/opt/gopath/src/github.com/opencontainers/runc'

  # We're going to compile go and run a concurrent system, so give ourselves
  # some extra resources. Nomad will have trouble working correctly with <2
  # CPUs so we should use at least that many.
  cpus = 2
  memory = 2048

  vmCfg.vm.provider "virtualbox" do |v|
    v.memory = memory
    v.cpus = cpus
  end

  ["vmware_fusion", "vmware_workstation"].each do |p|
    vmCfg.vm.provider p do |v|
      v.gui = false
      v.memory = memory
      v.cpus = cpus
    end
  end
  return vmCfg
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  1.upto(1) do |n|
    vmName = "runc-%02d" % [n]
    config.vm.define vmName, autostart: (n == 1 ? true : false), primary: (n == 1 ? true : false) do |vmCfg|
      vmCfg.vm.hostname = vmName
      vmCfg = configureVM(vmCfg)
    end
  end
end
