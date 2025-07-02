
# Spark Standalone Cluster Installation (Single Node)

> **Machine IP**: `192.168.30.95`  
> **Spark Version**: `4.0.0`  
> **Cluster Mode**: Standalone on a single machine (both Master and Worker)

---

## 🔗 Official Links

- [Spark Downloads](https://spark.apache.org/downloads.html)
- [Direct Spark TGZ Download](https://www.apache.org/dyn/closer.lua/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz)

---

## 📦 Installation Steps

### 1. Install Java

```bash
sudo apt install openjdk-17-jdk -y
java --version
readlink -f $(which java)
```

### 2. Download and Set Up Spark

```bash
wget https://dlcdn.apache.org/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz
tar -xvf spark-4.0.0-bin-hadoop3.tgz
sudo mv spark-4.0.0-bin-hadoop3 /opt/spark
```

### 3. Install Python 3.9 and PySpark

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.9 python3.9-distutils -y
curl -sS https://bootstrap.pypa.io/get-pip.py | sudo python3.9
pip3.9 install pyspark
```

### 4. Configure Environment Variables

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/opt/spark
export PYSPARK_PYTHON=/usr/bin/python3.9
```

Add the following to `~/.bashrc`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3.9
```

```bash
source ~/.bashrc
```

### 5. Configure Spark Files

```bash
cd /opt/spark/conf
cp spark-env.sh.template spark-env.sh
cp workers.template workers
```

Edit `spark-env.sh`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_MASTER_HOST=192.168.30.95
```

Edit `workers`:

```bash
localhost
```

### 6. Prepare Logs Directory

```bash
sudo mkdir -p /opt/spark/logs
sudo chown -R $USER:$USER /opt/spark/logs
```

---

## 🚀 Start Spark Services

```bash
start-master.sh
start-worker.sh spark://192.168.30.95:7077
```

Check Java processes:

```bash
jps
# Output should include Master and Worker
```

Open Spark UI:

```
http://192.168.30.95:8080
```

---

## 🧪 Test Spark Jobs

### JAR - Client Mode

```bash
spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.30.95:7077 --deploy-mode client /opt/spark/examples/jars/spark-examples_2.13-4.0.0.jar 10
```

- Driver: Host machine (95)
- Executors: Worker (95)

### JAR - Cluster Mode

```bash
spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.30.95:7077 --deploy-mode cluster /opt/spark/examples/jars/spark-examples_2.13-4.0.0.jar 10
```

- Driver: Worker (95)
- Executors: Worker (95)

### PySpark - Client Mode

```bash
spark-submit --master spark://192.168.30.95:7077 --deploy-mode client /home/syed/huge_job.py
```

- Driver: Host machine (95)
- Executors: Worker (95)

> Cluster mode is **not supported** for PySpark in standalone.

---

## 🔍 Check Spark Version

```bash
spark-submit --version
```

---

## 📝 Summary

| Component        | Location (192.168.30.95) |
|------------------|--------------------------|
| Master           | ✅                        |
| Worker           | ✅                        |
| Driver (Client)  | ✅ (host)                 |
| Driver (Cluster) | ✅ (worker)               |
| Executors        | ✅ (worker)               |
