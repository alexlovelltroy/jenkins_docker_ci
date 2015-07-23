# -*- mode: ruby -*-
# vi: set ft=ruby :

# Yves Hwang 11.03.2015
# Test Jenkins and Jenkins job builder setup

$script= <<SCRIPT
apt-get update -qq 
# Install docker deps
apt-get install -y linux-image-generic-lts-trusty linux-headers-generic-lts-trusty

# Install docker
wget -qO- https://get.docker.com/gpg | apt-key add -
wget -qO- https://get.docker.com/ | sh

usermod -a -G docker vagrant
sed -e 's/DOCKER_OPTS=/DOCKER_OPTS=\"-H 0.0.0.0:4243\"/g' /etc/init/docker.conf > /vagrant/docker.conf.sed
cp /vagrant/docker.conf.sed /etc/init/docker.conf
rm -f /vagrant/docker.conf.sed
service docker start

# Add the jenkins repo
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | apt-key add -
echo "deb http://pkg.jenkins-ci.org/debian binary/" >> /etc/apt/sources.list

# Install jenkins and the dependencies
apt-get update -qq 
apt-get install -y git python-pip devscripts debhelper checkinstall curl rst2pdf jenkins

# Install jenkins-job-builder
pip install jenkins-job-builder

# Install our jenkins backup
cd /
tar -jxf /vagrant/jenkins.tar.bz2

# start Jenkins the first time
service jenkins start

# Give it a chance to get going
sleep 30

wget http://localhost:8080/jnlpJars/jenkins-cli.jar
# WE DON'T NEED THE COMMENTED OUT PLUGINS
#java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin docker-plugin -deploy
#java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin docker-build-publish -deploy
#
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin docker-build-step -deploy
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin git -deploy
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin thinBackup -deploy
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin ghprb -deploy
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin token-macro -deploy
java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin junit -deploy
java -jar jenkins-cli.jar -s http://localhost:8080/ safe-restart;

# Give it a chance to restart
sleep 30

UPDATE_LIST=$( java -jar jenkins-cli.jar -s http://localhost:8080/ list-plugins | grep -e ')$' | awk '{ print $1 }' ); 
if [ ! -z "${UPDATE_LIST}" ]; then 
    echo Updating Jenkins Plugins: ${UPDATE_LIST}; 
    java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin ${UPDATE_LIST};
fi

java -jar jenkins-cli.jar -s http://localhost:8080/ safe-restart;

#cd /vagrant
#curl -X POST -d @localhost_test_cred http://localhost:8080/credential-store/domain/_/createCredentials -v
#service jenkins restart
service docker restart
SCRIPT


Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.provision :shell, :inline => $script
  config.vm.network "forwarded_port", guest: 8080, host: 38080
  config.vm.network "forwarded_port", guest: 4243, host: 4243
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--name", "jenkins_docker", "--memory", "2048"]
  end
end
