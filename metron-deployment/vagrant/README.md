# Metron Devbox Setup Guide
This guide can be used to deploy an Ambari-managed Hadoop cluster containing Metron services using Ansible. The Ansible playbooks target RHEL/CentOS 6.x operating systems. The host will require at least 8 GB of RAM and a fair amount of patience.

> Please note that in order to execute the job well, we actually tuned the RAM allocation from **8G** to **10G** in the *Vagrantfile* under the project

## Prerequisites

The computer used to deploy Apache Metron will need to have the following components installed.

 - [Ansible](https://github.com/ansible/ansible) (2.0.0.2 or 2.2.2.0)
 - [Docker](https://www.docker.com/community-edition)
 - [Vagrant](https://www.vagrantup.com) 1.8+
 - [Vagrant Hostmanager Plugin](https://github.com/devopsgroup-io/vagrant-hostmanager)
 - [Virtualbox](https://virtualbox.org) 5.0+
 - Python 2.7
 - Maven 3.3.9
 
### Mac Specific quickpass
If you are on Mac, you could *brew install* the required packages using following command and directly go to the ***Actual Kickoff Run*** section:

```
brew cask install vagrant virtualbox java docker
brew install maven@3.3 git
pip install ansible==2.2.2.0
vagrant plugin install vagrant-hostmanager
open /Applications/Docker.app
```
> As none Mac environment hasn't been proved yet, the following instructions could be change based on your situation, just make sure the required components are installed in your system, then you could proceed to ***Actual Kickoff Run*** section

### Install Python 
In you Centos system, install python using the following commands:

```shell
yum install epel-release -y
yum update -y
yum install vim tmux htop -y
yum install git wget curl rpm tar unzip bzip2 wget createrepo yum-utils ntp python-pip psutils python-psutil ntp libffi-devel gcc openssl-devel -y
```

### Install Ansible
After installed the python pip, you could then use:

```Shell
pip install --upgrade pip
pip install requests
pip install ansible==2.2.2.0
```
to install ansible


### Install Java and Maven

* Install Java8

```Shell
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel -y

```
* Setup ```JAVA_HOME```:

```Shell
export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s_/jre/bin/java__")
```
* Download and install Maven

```
cd ~/
wget http://apache.volia.net/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar -zxf apache-maven-3.3.9-bin.tar.gz
mv apache-maven-3.3.9 /opt 
PATH=/opt/apache-maven-3.3.9/bin:$PATH
echo 'export PATH=/opt/apache-maven-3.3.9/bin:$PATH' > /etc/profile.d/maven.sh
chmod +x /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
```
### Install Vagrant & Virtualbox

* Install virtualbox dependencies:

```
yum -y install gcc dkms make qt libgomp patch 
yum -y install kernel-headers kernel-devel binutils glibc-headers glibc-devel font-forge
```
* Add VirtualBox to repo list:

```
cd /etc/yum.repo.d/
wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
```

* Install VirtualBox 5.1:

```
yum install -y VirtualBox-5.1
/sbin/rcvboxdrv setup
```

* Install Vagrant:

```
----------- For 64-bit machine -----------
# yum -y install https://releases.hashicorp.com/vagrant/1.9.6/vagrant_1.9.6_x86_64.rpm
----------- For 32-bit machine ----------- 
# yum -y install https://releases.hashicorp.com/vagrant/1.9.6/vagrant_1.9.6_i686.rpm
```

* Install Vagrant Plugin (host-manager):

```
vagrant plugin install vagrant-hostmanager
```

## Actual Kickoff Run

You could managed to installed the above listed requirements in many ways as long as you keep the specific version required. After the prerequisites been installed on your machine.

### Clone the repo from Github (or Unzip)

If the metron.zip file is provided, please just unzip the metron.zip into your dedicated location, otherwise clone it from a forked github:

```
git clone https://github.com/timmyraynor/metron.git
```


### Kick off the deployment:
Once you have the *metron* folder created and all codes pulled. Please go to `{path-to-metron-folder}/metron/metron-deployment/vagrant/full-dev-platform/` as:

```
cd <path-to-metron-folder>/metron/metron-deployment/vagrant/full-dev-platform/
```

Then kick off the vagrant build:

```
vagrant up
```

>If any errors appears during the build, please use:
```
vagrant provision
```
to rerun the provision process, you could kick off the provision process multiple times until it ends up to the success stage. 

Wait until the process end (around 1.5 hour to build if your maven build and ambari server bootstraping runs well) , usually like below:

```
 TASK [deployment-report : debug] ***********************************************
ok: [node1] => {
    "Success": [
        "Apache Metron deployed successfully",
        "   Metron          @ http://node1:5000",
        "   Ambari          @ http://node1:8080",
        "   Sensor Status   @ http://node1:2812",
        "   Zookeeper       @ node1:2181",
        "   Kafka           @ node1:6667",
        "For additional information, see https://metron.apache.org/'"
    ]
}

PLAY RECAP *********************************************************************
node1                      : ok=133  changed=56   unreachable=0    failed=0
```

### End points

The major endpoints already listed as above ending states, to access [*Metron UI*](http://node1:5000), just use ```http://node1:5000```, Metron UI is essentially a [Kibana](https://www.elastic.co/products/kibana) dashboard.

The following block displays common end points and the default credential to access the service:

```
Ambari       : http://node1:8080       
    user     : admin
    password : admin

Storm UI:
    login into Ambari, click the storm section, then quicklinks -> storm ui
    
Monit Sensors:   http://node1:2812
    user     :   admin
    password :   monit
    
Metron       :   http://node1:5000
```

You could use the [Monit sensor dashboard](http://node1:2812) to manage existing sensors. Please note that some of these sensors are sensor-stub, which actually generate random sensor info for the testing purpose.

You could use the [Metron](http://node1:5000) for monitoring the events go through.

### Vagrant Extra

As you might noticed the whole VM is built using Vagrant, thus you could ssh into the machine using:

```
vagrant ssh
```

When you are in the working directory (in this case the `<...>/full-dev-platform/` folder.

As the Kafka instance is running inside the VM, you could see if the messages sent by the sensors exists in Kafka topics using following command under the **kafka bin folder**:

```
./kafka-console-consumer.sh --topic bro --zookeeper localhost:2181
```



