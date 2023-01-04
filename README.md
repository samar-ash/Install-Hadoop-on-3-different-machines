# Install-Hadoop-on-3-different-machines
1.	PREPERATION

First, create 3 virtual machines and install Ubuntu OS on them. While installing ubuntu OS on virtual machines, create both test and hduser OS users.
Then assign IP to each of them. I used the following IP addresses: 192.168.6.101 for master worker, and 192.168.56.102, 192.168.56.103 for two slave workers.
The Network setting could be as follows:
![image](https://user-images.githubusercontent.com/15922299/210484433-649f97ea-caf7-407a-8fdd-54391b7681b4.png)
![image](https://user-images.githubusercontent.com/15922299/210484511-69c82687-2079-40ff-bc80-76baf996c087.png)
![image](https://user-images.githubusercontent.com/15922299/210484538-a5bfe62c-fe31-4335-9f36-d0e6b8d1e7fa.png)
![image](https://user-images.githubusercontent.com/15922299/210484568-0a5f156d-1df7-47e6-8bc2-1bb075003672.png)
![image](https://user-images.githubusercontent.com/15922299/210484579-37971e0f-e1b6-4d4d-85a3-9c4ae939e193.png)
2.	INSTALLATION

After the master and slave nodes are getting ready, it is time to run the script below. The script can be used on both master and slaves installations with a simple question from the beginning of script which determines if it is Master or Slave Node. Also, it asks for IPs of the workers and save them to the variables for further use.

⸻⸻⸻⸻⸻⸻⸻⸻⸻⸻⸻⸻⸻⸻—


#!/bin/bash


username="hduser"
echo "Master or Slave(m/s)?:"
read worker

echo "What is the IP address  of master?: "
read master_ip

echo "What is the IP address  of slave1?: "
read slave1_ip

echo "What is the IP address  of slave2?: "
read slave2_ip

HOSTNAMEMASTER="hadoop-master" 
HOSTNAMESLAVEONE="hadoop-slaveone"
HOSTNAMESLAVETWO="hadoop-slavetwo"

echo  "$master_ip $HOSTNAMEMASTER" >> /etc/hosts
echo  "$slave1_ip  $HOSTNAMESLAVEONE " >> /etc/hosts
echo  "$slave2_ip $HOSTNAMESLAVETWO " >> /etc/hosts
cat /etc/hosts

# Add variable for $username, then use it everywhere 
username="hduser"

#Hadoop needs java to be installed so the next step is Installing Java:

#Install JAva
if type -p java; then
echo found java executable in PATH
_java=java
echo 'export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"' >>
~/.bashrc
elif [[ -n "$JAVA_HOME" ]] && [[ -x "$JAVA_HOME/bin/java" ]]; then echo found java executable in JAVA_HOME
_java="$JAVA_HOME/bin/java" else
echo "no java"
sudo apt install openjdk-8-jdk
echo 'export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"' >>
~/.bashrc fi

if [[ "$_java" ]]; then
version=$("$_java" -version 2>&1 | awk -F '"' '/version/ {print $2}') echo version ${version:0:3}
if [[ ${version:0:3} == 1.8 ]]; then echo version is ok 1.8
else
echo version is not 1.8
sudo apt-get autoremove java-common sudo apt install openjdk-8-jdk
echo 'export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"' >>
~/.bashrc
 
fi
fi

#SSH
sudo apt-get remove openssh-server openssh-client sudo apt-get update
sudo apt-get install openssh-server openssh-client sudo ufw allow 22
sudo systemctl restart ssh sudo apt-get install ssh sudo apt-get install rsync

# Generate Keys for secure communication:
sudo -u $username ssh-keygen -t rsa -P ""
sudo -u $username cat /home/$username/.ssh/id_rsa.pub >> /home/
$username/.ssh/authorized_keys

# Disable IPv6
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf echo "net.ipv6.conf.default.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf echo "net.ipv6.conf.lo.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
sudo -u $username cat /proc/sys/net/ipv6/conf/all/disable_ipv6 # Downloading Hadoop from Internet
if [ -d "/usr/local/hadoop" ]; then
echo "Hadoop is already downloaded."
else
echo "Hadoop is not downloaded from the Internet. Beginning to
download..."
sudo wget http://archive.apache.org/dist/hadoop/core/hadoop-3.3.4/ hadoop-3.3.4.tar.gz
sudo tar -xvf hadoop-3.3.4.tar.gz
sudo mv hadoop-3.3.4 /usr/local/hadoop
sudo chown -R $username:hadoop /usr/local/hadoop sudo -u $username chmod 777 /usr/local/hadoop
fi

# Creating tmp directory
sudo -u $username mkdir -p /usr/local/hadoop/tmp
sudo -u $username chown -R $username:hadoop /usr/local/hadoop/tmp sudo -u $username chmod 777 /usr/local/hadoop/etc/hadoop

echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" | sudo tee -a / usr/local/hadoop/etc/hadoop/hadoop-env.sh

echo "export HADOOP_HOME=/usr/local/hadoop" | sudo tee -a ~/.bashrc
 
echo "export PATH=\$PATH:\$HADOOP_HOME/sbin:\$HADOOP_HOME/bin" | sudo tee -a ~/.bashrc
echo "export HADOOP_COMMON_HOME=\$HADOOP_HOME" | sudo tee -a
~/.bashrc
echo "export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop" | sudo tee
-a ~/.bashrc
echo "export HADOOP_HDFS_HOME=\$HADOOP_HOME" | sudo tee -a
~/.bashrc
echo "export HADOOP_MAPRED_HOME=\$HADOOP_HOME" | sudo tee -a
~/.bashrc
echo "export HADOOP_YARN_HOME=\$HADOOP_HOME" | sudo tee -a
~/.bashrc


# Edit hadoop core file
sed -i '/<configuration>/d' /usr/local/hadoop/etc/hadoop/core-site.xml sed -i '/<\/configuration>/d' /usr/local/hadoop/etc/hadoop/core-site.xml

echo "<configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/core- site.xml
echo "<property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/core-site.xml echo "  <name>fs.defaultFS</name>" | sudo tee -a /usr/local/hadoop/etc/ hadoop/core-site.xml
echo "  <value>hdfs://$HOSTNAMEMASTER:9000</value>" | sudo tee -a / usr/local/hadoop/etc/hadoop/core-site.xml
echo "</property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/core-site.xml echo "</configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/core- site.xml

# Edit hadoop Map Reduce file
sed -i '/<configuration>/d' /usr/local/hadoop/etc/hadoop/mapred-site.xml sed -i '/<\/configuration>/d' /usr/local/hadoop/etc/hadoop/mapred-site.xml

echo "<configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/mapred- site.xml
echo "<property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/mapred-site.xml echo "  <name>mapred.job.tracker</name>" | sudo tee -a /usr/local/hadoop/ etc/hadoop/mapred-site.xml
echo "  <value>$HOSTNAMEMASTER:9001</value>" | sudo tee -a /usr/local/ hadoop/etc/hadoop/mapred-site.xml
echo "</property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/mapred-site.xml echo "</configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/mapred- site.xml

# Edits HDFS file
sed -i '/<configuration>/d' /usr/local/hadoop/etc/hadoop/hdfs-site.xml sed -i '/<\/configuration>/d' /usr/local/hadoop/etc/hadoop/hdfs-site.xml
 
echo "<configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs- site.xml
echo "<property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml echo "  <name>dfs.namenode.name.dir</name>" | sudo tee -a /usr/local/ hadoop/etc/hadoop/hdfs-site.xml
echo " <value>file:/usr/local/hadoop/yarn_data/hdfs/namenode</value>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml
echo "</property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml echo "<property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml echo "	<name>dfs.datanode.data.dir</name>" | sudo tee -a /usr/local/ hadoop/etc/hadoop/hdfs-site.xml
echo "	<value>file:/usr/local/hadoop/yarn_data/hdfs/datanode</value>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml
echo "</property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml echo "<property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml echo "	<name>dfs.replication</name>" | sudo tee -a /usr/local/hadoop/etc/ hadoop/hdfs-site.xml
echo "	<value>1</value>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs- site.xml
echo "</property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs-site.xml echo "</configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/hdfs- site.xml

# Edit hadoop Yarn file
sed -i '/<configuration>/d' /usr/local/hadoop/etc/hadoop/yarn-site.xml sed -i '/<\/configuration>/d' /usr/local/hadoop/etc/hadoop/yarn-site.xml

echo "<configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/yarn- site.xml
echo "<property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/yarn-site.xml echo "	<name>yarn.resourcemanager.hostname</name>" | sudo tee -a /usr/ local/hadoop/etc/hadoop/yarn-site.xml
echo "	<value>$HOSTNAMEMASTER</value>" | sudo tee -a /usr/local/ hadoop/etc/hadoop/yarn-site.xml
echo "</property>" | sudo tee -a /usr/local/hadoop/etc/hadoop/yarn-site.xml echo "</configuration>" | sudo tee -a /usr/local/hadoop/etc/hadoop/yarn- site.xml

if [[ $ISMASTER == 1 ]]; then echo Master initializing… read ip

ssh-copy-id -i ~/.ssh/id_rsa.pub hduser@hadoop-slaveone ssh-copy-id -i ~/.ssh/id_rsa.pub hduser@hadoop-slavetwo

echo "$HOSTNAMESLAVEONE" | sudo tee -a /usr/local/hadoop/etc/hadoop/
 
workers
echo "$HOSTNAMESLAVETWO" | sudo tee -a /usr/local/hadoop/etc/hadoop/ workers

source ~/.bashrc
hdfs namenode -format start-dfs.sh
start-yarn.sh jps

else
echo Slave initializing… read ip

ssh-copy-id -i ~/.ssh/id_rsa.pub hduser@hadoop-master fi
 
