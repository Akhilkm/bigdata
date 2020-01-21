# Create a 3 node apache spark cluster over yarn

### Prerequisites
```
3 running ubuntu machines with the below specification
    * 4 cores
    * 8 GB RAM
    * 64-bit (x86)
    * Inter communication enabled
```

```
Add hostname and ip for all the machines 

hostnamectl set-hostname hadoop-server<number>
hostname -F /etc/hostname

echo """
<Ip of hadoop-server1> hadoop-server1
<Ip of hadoop-server2> hadoop-server2
<Ip of hadoop-server3> hadoop-server3
""" > /etc/hosts
```

```
Set Locale and add JAVA_HOME environment variable

echo """
LANGUAGE=\"en_US.UTF-8\"
LANG=\"en_US.UTF-8\"
LC_ALL=\"en_US.UTF-8\"
JAVA_HOME=\"/usr/lib/jvm/java-8-openjdk-amd64\"
""" >> /etc/environment

exit and reconnect to all the machines
```

```
Update the OS and install java8

apt-get update && apt-get -y upgrade
apt-get install openjdk-8-jdk-headless
```