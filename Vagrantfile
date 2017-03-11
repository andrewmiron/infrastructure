# -*- mode: ruby -*-
# vi: set ft=ruby :

$ci_script = <<SCRIPT
echo Setting variables
PROJECT=am_university_repo
VERSION=`grep DISTRIB_RELEASE /etc/lsb-release | cut -d'=' -f2`
PLUGINS_DIR="/var/lib/jenkins/plugins"
if [ $VERSION = "14.04" ]; then
  USERNAME="vagrant"
else
  USERNAME="ubuntu"
fi
echo Provision the CI VM ...
echo Update and upgrade ...
apt-get update && apt-get upgrade -qy
apt-get autoremove -qy
apt-get install git sshpass unzip -qy
echo Configuring GIT ...
echo "[user]" > /home/$USERNAME/.gitconfig
echo "email = admin@localhost.com" >> /home/$USERNAME/.gitconfig
echo "name = admin" >> /home/$USERNAME/.gitconfig
chown -R $USERNAME:$USERNAME /home/$USERNAME/.gitconfig
echo Creating GIT repository...
rm -rf /vagrant/$PROJECT.git
git init --bare /vagrant/$PROJECT.git
echo Installing Java and Maven ...
if [ $VERSION = "14.04" ]; then
  apt-get install openjdk-7-jdk maven2 -qy
else
  apt-get install openjdk-8-jdk maven -qy
fi
echo Installing Jenkins CI ...
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
apt-get update
apt-get install jenkins -qy
echo 'JAVA_ARGS="-Djava.awt.headless=true -Djenkins.install.runSetupWizard=false"' >> /etc/default/jenkins
if [ $VERSION = "14.04" ]; then
  mkdir -p $PLUGINS_DIR
else
  mkdir -p $PLUGINS_DIR
fi
echo Install Jenkins Plugins ...
wget -O /var/lib/jenkins/plugins/bouncycastle-api.hpi https://updates.jenkins-ci.org/latest/bouncycastle-api.hpi
wget -O /var/lib/jenkins/plugins/credentials.hpi https://updates.jenkins-ci.org/latest/credentials.hpi
wget -O /var/lib/jenkins/plugins/display-url-api.hpi https://updates.jenkins-ci.org/latest/display-url-api.hpi
wget -O /var/lib/jenkins/plugins/git-client.hpi https://updates.jenkins-ci.org/latest/git-client.hpi
wget -O /var/lib/jenkins/plugins/git.hpi https://updates.jenkins-ci.org/latest/git.hpi
wget -O /var/lib/jenkins/plugins/junit.hpi https://updates.jenkins-ci.org/latest/junit.hpi
wget -O /var/lib/jenkins/plugins/mailer.hpi https://updates.jenkins-ci.org/latest/mailer.hpi
wget -O /var/lib/jenkins/plugins/matrix-project.hpi https://updates.jenkins-ci.org/latest/matrix-project.hpi
wget -O /var/lib/jenkins/plugins/workflow-scm-step.hpi https://updates.jenkins-ci.org/latest/workflow-scm-step.hpi
wget -O /var/lib/jenkins/plugins/workflow-scm-api.hpi https://updates.jenkins-ci.org/latest/workflow-scm-api.hpi
wget -O /var/lib/jenkins/plugins/scm-api.hpi https://updates.jenkins-ci.org/latest/scm-api.hpi
wget -O /var/lib/jenkins/plugins/script-security.hpi https://updates.jenkins-ci.org/latest/script-security.hpi
wget -O /var/lib/jenkins/plugins/ssh-credentials.hpi https://updates.jenkins-ci.org/latest/ssh-credentials.hpi
wget -O /var/lib/jenkins/plugins/structs.hpi https://updates.jenkins-ci.org/latest/structs.hpi
wget -O /var/lib/jenkins/plugins/workflow-basic-steps.hpi https://updates.jenkins-ci.org/latest/workflow-basic-steps.hpi
wget -O /var/lib/jenkins/plugins/resource-disposer.hpi https://updates.jenkins-ci.org/latest/resource-disposer.hpi
wget -O /var/lib/jenkins/plugins/ws-cleanup.hpi https://updates.jenkins-ci.org/latest/ws-cleanup.hpi
wget -O /var/lib/jenkins/plugins/config-file-provider.hpi https://updates.jenkins-ci.org/latest/config-file-provider.hpi
wget -O /var/lib/jenkins/plugins/token-macro.hpi https://updates.jenkins-ci.org/latest/token-macro.hpi
wget -O /var/lib/jenkins/plugins/workflow-api.hpi https://updates.jenkins-ci.org/latest/workflow-api.hpi
wget -O /var/lib/jenkins/plugins/workflow-step-api.hpi https://updates.jenkins-ci.org/latest/workflow-step-api.hpi
wget -O /var/lib/jenkins/plugins/maven-plugin.hpi https://updates.jenkins-ci.org/latest/maven-plugin.hpi
wget -O /var/lib/jenkins/plugins/javadoc.hpi https://updates.jenkins-ci.org/latest/javadoc.hpi
if [ $VERSION = "14.04" ]; then
  chown -R jenkins:jenkins $PLUGINS_DIR
  cp /vagrant/configs/$VERSION/* /var/lib/jenkins
else
  chown -R jenkins:jenkins $PLUGINS_DIR
  cp /vagrant/configs/$VERSION/* /var/lib/jenkins
fi
service jenkins restart
echo Installing Artifactory...
if [ -f /var/lib/artifactory/bin/installService.sh ]; then
  echo Artifactory is already installed
else
  cd /var/lib
  wget https://bintray.com/artifact/download/jfrog/artifactory/artifactory-3.9.4.zip
  unzip artifactory-3.9.4.zip
  mv artifactory-3.9.4 artifactory
  sudo -u root /var/lib/artifactory/bin/installService.sh
  service artifactory start
  rm artifactory-3.9.4.zip
fi
echo Done. 
SCRIPT

$app_script = <<SCRIPT
echo Provision the APP VM ...
echo Update and upgrade ...
apt-get update && apt-get upgrade -qy
apt-get autoremove -qy
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  # config.vm.box = "ubuntu/xenial64"

  config.vm.define "ci" do |ci|
    ci.vm.hostname = "am-ci"
    ci.vm.provision :shell, :inline => $ci_script
    ci.vm.network "forwarded_port", guest: 22, host: 8022
    ci.vm.network "forwarded_port", guest: 8080, host: 8080
    ci.vm.network "forwarded_port", guest: 8081, host: 8081
    ci.vm.provider "virtualbox" do |vm|
      vm.customize [
                    'modifyvm', :id,
                    '--memory', '2048',
                    '--cpus', '2',
                  ]
    end
  end
  config.vm.define "app" do |app|
    app.vm.hostname = "am-app"
    app.vm.provision :shell, :inline => $app_script
    app.vm.network "forwarded_port", guest: 22, host: 8122
    app.vm.network "forwarded_port", guest: 9090, host: 9090
    app.vm.provider "virtualbox" do |vm|
      vm.customize [
                    'modifyvm', :id,
                    '--memory', '512',
                  ]
    end
  end
end
