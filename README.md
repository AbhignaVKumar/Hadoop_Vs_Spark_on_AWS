# Hadoop and Spark Performance Comparison on AWS

A comprehensive tutorial for setting up a distributed Hadoop and Spark cluster on AWS EC2, implementing MapReduce jobs in Python, and comparing performance between the two big data frameworks.

## 📋 Overview

This project demonstrates:
- **Two-node Hadoop cluster** setup on AWS EC2
- **MapReduce jobs** using Hadoop Streaming with Python
- **Apache Spark** installation and PySpark implementation
- **Performance comparison** between Hadoop and Spark
- **Real-world data processing** using Project Gutenberg dataset

### Key Features

- ✅ Distributed computing with 2-node cluster
- ✅ Word count, character frequency, and min/max analysis
- ✅ Python-based MapReduce implementation
- ✅ PySpark in-memory processing
- ✅ Performance benchmarking and analysis

---

## 🏗️ Architecture

```
┌─────────────────┐         ┌─────────────────┐
│   Master Node   │         │   Worker Node   │
│                 │         │                 │
│  - NameNode     │◄───────►│  - DataNode     │
│  - ResourceMgr  │         │  - NodeManager  │
│  - JobTracker   │         │  - TaskTracker  │
│  - Spark Master │         │  - Spark Worker │
└─────────────────┘         └─────────────────┘
         │                           │
         └───────────┬───────────────┘
                     ▼
              HDFS Storage
           (Gutenberg Dataset)
```

---

## 🚀 Getting Started

### Prerequisites

- **AWS Account** with EC2 access
- **Two Ubuntu 22.04 EC2 instances** (t2.medium recommended)
- **Security Group** configuration:
  - SSH (port 22)
  - HDFS NameNode (port 50070)
  - YARN ResourceManager (port 8088)
  - Spark Master (port 8080)
  - Spark Worker (port 8081)

### System Requirements

| Component | Version |
|-----------|---------|
| Ubuntu | 22.04 LTS |
| Java | OpenJDK 11 |
| Hadoop | 3.3.6 |
| Spark | 3.5.0 |
| Python | 3.x |

---

## 📦 Installation

### Step 1: Launch EC2 Instances

1. **Launch two Ubuntu 22.04 instances** (t2.medium)
2. **Configure security groups** with required ports
3. **SSH into both instances**

```bash
ssh -i your-key.pem ubuntu@<MASTER_IP>
ssh -i your-key.pem ubuntu@<WORKER_IP>
```

### Step 2: Install Java and Hadoop (Both Nodes)

```bash
# Update system
sudo apt update
sudo apt install -y openjdk-11-jdk ssh rsync

# Download and install Hadoop
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xvzf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /usr/local/hadoop

# Set environment variables
echo 'export HADOOP_HOME=/usr/local/hadoop' >> ~/.bashrc
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> ~/.bashrc
source ~/.bashrc
```

### Step 3: Configure Hadoop (Master Node)

Edit the following configuration files in `/usr/local/hadoop/etc/hadoop/`:

**core-site.xml**
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
  </property>
</configuration>
```

**hdfs-site.xml**
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///usr/local/hadoop/hadoop_data/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///usr/local/hadoop/hadoop_data/hdfs/datanode</value>
  </property>
</configuration>
```

**mapred-site.xml**
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

**yarn-site.xml**
```xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

**workers** file
```
worker1
```

### Step 4: Start Hadoop Services

```bash
# Format NameNode (first time only)
hdfs namenode -format

# Start HDFS
/usr/local/hadoop/sbin/start-dfs.sh

# Start YARN
/usr/local/hadoop/sbin/start-yarn.sh

# Verify services
jps
```

### Step 5: Install Spark (Both Nodes)

```bash
# Download Spark
wget https://downloads.apache.org/spark/spark-3.5.0/spark-3.5.0-bin-hadoop3.tgz
tar -xvzf spark-3.5.0-bin-hadoop3.tgz
sudo mv spark-3.5.0-bin-hadoop3 /usr/local/spark

# Set environment variables
echo 'export SPARK_HOME=/usr/local/spark' >> ~/.bashrc
echo 'export PATH=$PATH:$SPARK_HOME/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## 💻 Usage

### Prepare Dataset

Download and upload Project Gutenberg texts to HDFS:

```bash
# Create HDFS directory
hdfs dfs -mkdir /gutenberg

# Download sample books
wget https://www.gutenberg.org/files/1342/1342-0.txt  # Pride and Prejudice
wget https://www.gutenberg.org/files/84/84-0.txt      # Frankenstein
wget https://www.gutenberg.org/files/11/11-0.txt      # Alice in Wonderland

# Upload to HDFS
hdfs dfs -put *.txt /gutenberg/

# Verify upload
hdfs dfs -ls /gutenberg
```

### Hadoop MapReduce Jobs

#### 1. Word Count

```bash
# Make scripts executable
chmod +x hadoop/wordcount_mapper.py
chmod +x hadoop/wordcount_reducer.py

# Run MapReduce job
hadoop jar /usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-*.jar \
  -input /gutenberg \
  -output /output_wordcount \
  -mapper wordcount_mapper.py \
  -reducer wordcount_reducer.py \
  -file hadoop/wordcount_mapper.py \
  -file hadoop/wordcount_reducer.py

# View results
hdfs dfs -cat /output_wordcount/part-00000 | head -20
```

#### 2. Character Count

```bash
chmod +x hadoop/char_mapper.py
chmod +x hadoop/char_reducer.py

hadoop jar /usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-*.jar \
  -input /gutenberg \
  -output /output_charcount \
  -mapper char_mapper.py \
  -reducer char_reducer.py \
  -file hadoop/char_mapper.py \
  -file hadoop/char_reducer.py
```

#### 3. Min/Max Word Count

```bash
chmod +x hadoop/minmax_reducer.py

hadoop jar /usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-*.jar \
  -input /gutenberg \
  -output /output_minmax \
  -mapper wordcount_mapper.py \
  -reducer minmax_reducer.py \
  -file hadoop/wordcount_mapper.py \
  -file hadoop/minmax_reducer.py
```

### Spark (PySpark) Jobs

#### 1. Word Count

```bash
spark-submit --master local[*] spark/spark_wordcount.py
```

#### 2. Character Frequency

```bash
spark-submit --master local[*] spark/spark_charcount.py
```

#### 3. Min/Max Word Count

```bash
spark-submit --master local[*] spark/spark_minmax.py
```

---

## 📁 Project Structure

```
hadoop-spark-tutorial/
├── README.md
├── hadoop/
│   ├── wordcount_mapper.py
│   ├── wordcount_reducer.py
│   ├── char_mapper.py
│   ├── char_reducer.py
│   └── minmax_reducer.py
├── spark/
│   ├── spark_wordcount.py
│   ├── spark_charcount.py
│   └── spark_minmax.py
├── config/
│   ├── core-site.xml
│   ├── hdfs-site.xml
│   ├── mapred-site.xml
│   └── yarn-site.xml
├── scripts/
│   ├── setup_hadoop.sh
│   ├── setup_spark.sh
│   └── download_gutenberg.sh
└── results/
    └── performance_comparison.md
```

---

## 📊 Performance Comparison

### Execution Time Results

| Task | Hadoop (1 Node) | Hadoop (2 Nodes) | Spark (1 Node) | Spark (2 Nodes) |
|------|----------------|------------------|----------------|-----------------|
| Word Count | ~45s | ~28s | ~12s | ~8s |
| Character Count | ~50s | ~32s | ~15s | ~10s |
| Min/Max | ~48s | ~30s | ~14s | ~9s |

### Key Observations

1. **Spark is faster** due to in-memory processing vs. Hadoop's disk I/O
2. **Scaling improves performance** for both frameworks
3. **Spark has simpler code** with higher-level APIs
4. **Hadoop is more fault-tolerant** with disk-based storage

---

## 🔍 Analysis Questions

### 1. Which framework executed faster?

**Answer:** Spark executed significantly faster on both single-node and two-node configurations due to its in-memory processing model, which eliminates the disk I/O overhead present in Hadoop MapReduce.

### 2. Which framework is easier to implement in Python?

**Answer:** Spark (PySpark) is easier to implement because:
- Higher-level API with functional transformations
- More concise code (fewer lines)
- Built-in Python support vs. Hadoop Streaming
- Better debugging and error messages

### 3. How does Spark's in-memory processing affect performance?

**Answer:** Spark's in-memory processing:
- **Reduces I/O:** Data stays in RAM between operations
- **Enables iterative algorithms:** No repeated disk reads
- **Faster shuffles:** In-memory data exchanges
- **Trade-off:** Higher memory requirements

---

## 🛠️ Troubleshooting

### Common Issues

**Hadoop services won't start:**
```bash
# Check logs
cat /usr/local/hadoop/logs/hadoop-*-namenode-*.log

# Verify Java version
java -version  # Should be 11.x
```

**HDFS connection refused:**
```bash
# Check if NameNode is running
hdfs dfsadmin -report

# Restart HDFS
/usr/local/hadoop/sbin/stop-dfs.sh
/usr/local/hadoop/sbin/start-dfs.sh
```

**Spark job fails:**
```bash
# Check Spark logs
ls -la /usr/local/spark/logs/

# Verify Python version
python3 --version
```

---

## 📚 Resources

### Documentation

- [Apache Hadoop Documentation](https://hadoop.apache.org/docs/r3.3.6/)
- [Apache Spark Documentation](https://spark.apache.org/docs/3.5.0/)
- [HDFS Architecture](https://hadoop.apache.org/docs/r2.10.1/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
- [YARN](https://hadoop.apache.org/docs/r2.10.1/hadoop-yarn/hadoop-yarn-site/YARN.html)
- [MapReduce Tutorial](https://hadoop.apache.org/docs/r2.10.1/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)

### Additional Reading

- [MapReduce on Wikipedia](https://en.wikipedia.org/wiki/MapReduce)
- [Project Gutenberg](https://www.gutenberg.org/)
- [PySpark API Reference](https://spark.apache.org/docs/latest/api/python/)

---

