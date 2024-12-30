
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


### **1. Download the Correct Spark Version**
Go to the official [Apache Spark Downloads](https://spark.apache.org/downloads.html) page and get the most recent version compatible with Hadoop 3.x. 

Alternatively, you can download Spark from the Apache mirror archives directly. Here’s a link to the Apache mirror for Spark 3.4.1:

```bash
wget https://archive.apache.org/dist/spark/spark-3.4.1/spark-3.4.1-bin-hadoop3.tgz
```

### **2. Extract and Move Spark**
After successfully downloading the correct version, follow these commands:

```bash
sudo tar -xvzf spark-3.4.1-bin-hadoop3.tgz -C /usr/local
sudo mv /usr/local/spark-3.4.1-bin-hadoop3 /usr/local/spark
sudo chown -R hduser:hadoop /usr/local/spark
```

### **3. Set Spark Environment Variables**
Edit your `~/.bashrc` to set the Spark environment variables:

```bash
nano ~/.bashrc
```

Add the following lines:

```bash
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
```

### **4. Reload the `~/.bashrc` File**
After editing the `.bashrc` file, run the following command to apply the changes:

```bash
source ~/.bashrc
```

### **5. Test Spark Installation**
Run the following command to start the PySpark shell:

```bash
pyspark
```

### **What Works Without YARN and MapReduce?**
- Spark in standalone mode for:
  - **ETL operations**.
  - **Data analysis** using PySpark or Scala.
  - **Machine learning tasks** with MLlib.
  - Running jobs on HDFS or local storage.

- Hadoop HDFS for:
  - Managing data storage in a distributed file system.
  - Performing basic data ingestion and retrieval.

---

### **Recommendation for Free Tier**
Since Free Tier AWS provides limited resources (1 vCPU, 1 GB RAM for t2.micro instances), it’s better to keep things simple:

1. **Focus on Spark in Standalone Mode**:
   - Skip configuring **yarn-site.xml** and **mapred-site.xml**.
   - Use Spark for data processing and parallel computations.

2. **Use HDFS with Basic Hadoop Configurations**:
   - Keep **core-site.xml** and **hdfs-site.xml** for file system management.

3. **Upgrade When Needed**:
   - If you later move to a larger AWS instance or a cluster setup, you can revisit configuring YARN and MapReduce for more advanced workloads.

---

### **Updated Instructions to Include yarn-site.xml and mapred-site.xml**

#### **1. Edit yarn-site.xml**
```bash
nano $HADOOP_CONF_DIR/yarn-site.xml
```

Add the following configuration:
```xml
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
</configuration>
```

---

#### **2. Edit mapred-site.xml**
The **mapred-site.xml** file does not exist by default. Copy the template first:
```bash
cp $HADOOP_CONF_DIR/mapred-site.xml.template $HADOOP_CONF_DIR/mapred-site.xml
```

Now edit the file:
```bash
nano $HADOOP_CONF_DIR/mapred-site.xml
```

Add the following configuration:
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>localhost:10020</value>
  </property>
</configuration>
```

---


---
# Please Remember to change jdk version 8 before start working Hive

### **Step 1: Use the Correct Hive Download Link**
Instead of the previous link, use this updated and reliable link from the **Apache Archive**:

```bash
wget https://archive.apache.org/dist/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz
```

---

### **Step 2: Verify the File After Download**
After the download completes, confirm the file exists by listing the files in your directory:
```bash
ls
```

You should see the `apache-hive-3.1.3-bin.tar.gz` file.

---

### **Step 3: Continue the Installation**
If the file downloads successfully, extract it and proceed with the remaining installation steps:
```bash
sudo tar -xvzf apache-hive-3.1.3-bin.tar.gz -C /usr/local
sudo mv /usr/local/apache-hive-3.1.3-bin /usr/local/hive
sudo chown -R hduser:hadoop /usr/local/hive
```

---


### **Step 3: Set Hive Environment Variables**
Edit the `~/.bashrc` file to include Hive environment variables:
```bash
nano ~/.bashrc
```

Add the following lines:
```bash
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
```

Apply the changes:
```bash
source ~/.bashrc
```

---

### **Step 4: Configure Hive**
Hive requires configuration files for its setup. Navigate to the Hive configuration directory and create these files:
```bash
cd $HIVE_HOME/conf
cp hive-default.xml.template hive-site.xml
```

Edit `hive-site.xml`:
```bash
nano hive-site.xml
```

Add the following basic configuration:

```xml
<configuration>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.apache.derby.jdbc.EmbeddedDriver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>APP</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>mine</value>
    </property>
</configuration>
```

---

### **Step 5: Create Hive Warehouse Directory in HDFS**
Hive stores its tables and data in HDFS. Create the directory and set permissions:
```bash
hdfs dfs -mkdir /user/hive/warehouse
hdfs dfs -chmod -R 777 /user/hive/warehouse
```

---

### **Step 6: Initialize Hive Metastore**
Run the following command to initialize the metastore:
```bash
schematool -initSchema -dbType derby
```

---

### **Step 7: Verify Hive Installation**
Start the Hive shell:
```bash
hive
```

Run the following command in the Hive shell to create a test table:
```sql
CREATE TABLE test_table (id INT, name STRING);
SHOW TABLES;
```

If the table appears in the output, Hive has been successfully installed.

---

### **Step 8: Optional – Use MySQL as Metastore**
If you plan to use Hive extensively, consider using **MySQL** instead of the embedded **Derby** database for the metastore. This is optional for small setups and Free Tier.

To use MySQL:
1. Install MySQL:
   ```bash
   sudo apt install mysql-server -y
   ```
2. Configure MySQL for Hive metastore and replace the database connection properties in `hive-site.xml`.

---





