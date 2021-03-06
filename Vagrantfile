# -*- mode: ruby -*-
# vi: set ft=ruby :

# The MIT License (MIT)
# 
# Copyright (c) 2017 Kevin Brightwell and Daniel Bailey
#
# Permission is hereby granted, free of charge, to any person obtaining a copy 
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights 
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

# This vagrantfile is used for creating a ubuntu xenial environment for 
# building the procsim project

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"

  cpu_count = ENV.has_key?("CPUS") ? ENV["CPUS"] : 2
  memory_count = ENV.has_key?("MEMORY") ? ENV["MEMORY"] : 2048

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = cpu_count
    vb.memory = memory_count
    
    vb.customize ["modifyvm", :id, "--uartmode1", "file", "procsim-xenial.log" ]
    
    vb.gui = true
    vb.customize ["modifyvm", :id, "--vram", "128"]
  end

  # Install/update prerequisite software:

  config.vm.provision "shell", privileged: true, inline: <<-EOF
    apt-add-repository ppa:george-edison55/cmake-3.x -y
    wget -O - http://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add -

    apt-get update -qq
    apt-get dist-upgrade -y -q 

    # For some reason, the box is broken with the locale
    localectl set-locale LANG="en_US.UTF-8"

  EOF

  config.vm.provision "shell", privileged: true, inline: <<-EOF
    echo "Installing build prequisisites: git, python3"
    apt-get install -q -y git 
    apt-get install -qq -y python3
  EOF

  config.vm.provision "shell", privileged: true, inline: <<-EOF
    echo "Installing build compiler toolchains: clang gcc-5"

    apt-get install -y clang-3.7 --force-yes

    apt-get install -q -y gcc g++ \
                          --force-yes
  EOF

  config.vm.provision "shell", privileged: true, inline: <<-EOF
    echo "Installing build dependencies: ccache, and ninja"
    
    apt-get install -qq -y bash-completion --reinstall
    apt-get install -qq -y ccache ninja-build \
                          --no-install-recommends
  EOF

  # Install VSCode 
  config.vm.provision "shell", privileged: true, inline: <<-EOF
    echo "Installing Visual Studio Code"
	
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
    sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
    
    apt-get update
    apt-get install -y code
  EOF
  
  config.vm.provision "shell", privileged: true, inline: <<-EOM
    echo "Creating alternatives for clang and gcc"

    update-alternatives --install /usr/bin/cc cc `which gcc-5` 30
    update-alternatives --install /usr/bin/c++ c++ `which g++-5` 30
    
    update-alternatives --install /usr/bin/cc cc `which clang-3.7` 40
    update-alternatives --install /usr/bin/c++ c++ `which clang++-3.7` 40
  EOM
  
  if ENV.has_key?("WORKSPACE")
    WORKSPACE = ENV["WORKSPACE"]
    config.vm.provision "shell", run: "always", inline: "echo 'Linking workspace: ~/workspace -> #{WORKSPACE}'"
    
    config.vm.synced_folder WORKSPACE, "/home/vagrant/workspace"
  end

  config.vm.provision "shell", privileged: true, inline: <<-EOF 
    apt-get install -y ubuntu-desktop \
                      unity-lens-applications \
                      unity-lens-files
  EOF

  # Do this last since it's best to make sure everyone else is updated first
  config.vm.provision "shell", privileged: true, inline: "apt-get dist-upgrade -y ; apt-get autoremove --purge -y"
  
  config.vm.provision :reload
end # end vagrant file 
