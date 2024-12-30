
---

### **Updated Apache Hadoop and Spark Installation Guide**

#### **1. Create EC2 Instance with Ubuntu 22.04 or 20.04**
Use the latest stable LTS version of Ubuntu instead of 18.04 for better security and updates.

---

#### **2. Create a New User with Root Permissions**
```bash
sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser
sudo usermod -aG sudo hduser
```

Switch to the new user:
```bash
su - hduser
```

---

#### **3. Install Java**
Install the latest version of OpenJDK:
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Verify Java installation:
```bash
java -version
```

Set the Java environment variables in `~/.bashrc`:
```bash
nano ~/.bashrc
```
Add the following lines:
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

Reload the `~/.bashrc` file:
```bash
source ~/.bashrc
```

---

#### **4. Set Up Passwordless SSH**
Generate an SSH key and configure passwordless SSH for the `hduser`:
```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
ssh localhost
```

---

#### **5. Install Hadoop**
Download the latest version of Hadoop from the [Apache Hadoop Downloads](https://hadoop.apache.org/releases.html):
```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
sudo tar -xvzf hadoop-3.3.6.tar.gz -C /usr/local
sudo mv /usr/local/hadoop-3.3.6 /usr/local/hadoop
sudo chown -R hduser:hadoop /usr/local/hadoop
sudo chmod -R 777 /usr/local/hadoop
```

Disable IPv6 in `/etc/sysctl.conf`:
```bash
sudo nano /etc/sysctl.conf
```
Add the following lines:
```bash
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
```

Reload the configuration:
```bash
sudo sysctl -p
```

Update Hadoop environment variables in `~/.bashrc`:
```bash
nano ~/.bashrc
```
Add:
```bash
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Reload the `~/.bashrc` file:
```bash
source ~/.bashrc
```

Configure `hadoop-env.sh`:
```bash
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
Set:
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true"
```

---

#### **6. Configure Hadoop**
Edit the following files:

**core-site.xml**:
```bash
nano $HADOOP_CONF_DIR/core-site.xml
```
Add:
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

**hdfs-site.xml**:
```bash
nano $HADOOP_CONF_DIR/hdfs-site.xml
```
Add:
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/yarn_data/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop/yarn_data/hdfs/datanode</value>
  </property>
</configuration>
```

Create required directories:
```bash
sudo mkdir -p /usr/local/hadoop/yarn_data/hdfs/namenode
sudo mkdir -p /usr/local/hadoop/yarn_data/hdfs/datanode
sudo chown -R hduser:hadoop /usr/local/hadoop/yarn_data
sudo chmod -R 700 /usr/local/hadoop/yarn_data
```

Format the Hadoop filesystem:
```bash
hdfs namenode -format
```

---

#### **7. Start Hadoop**
Start the HDFS and YARN services:
```bash
start-dfs.sh
start-yarn.sh
```

Verify the services:
- Namenode UI: `http://<your-ec2-public-ip>:9870`
- Resource Manager UI: `http://<your-ec2-public-ip>:8088`

---

