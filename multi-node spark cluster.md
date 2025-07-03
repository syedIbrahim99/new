# Spark Standalone Cluster Setup (Multi-Node)

## Official Documentation

- [Spark Downloads](https://spark.apache.org/downloads.html)
- [Spark 4.0.0 Direct Download (Hadoop 3)](https://www.apache.org/dyn/closer.lua/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz)
- [Archive Versions](https://archive.apache.org/dist/spark/)

## Machine Details

- **Machine A (Master)**: `192.168.30.95`
- **Machine B (Worker)**: `192.168.30.96`

---

## Machine A - Master Setup (`192.168.30.95`)

### Java Installation

```bash
sudo apt install openjdk-17-jdk -y
java --version
```

### Download and Install Spark

```bash
wget https://dlcdn.apache.org/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz
tar -xvf spark-4.0.0-bin-hadoop3.tgz
sudo mv spark-4.0.0-bin-hadoop3 /opt/spark
readlink -f $(which java)
```

### Python and PySpark Installation

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.9 python3.9-distutils -y
curl -sS https://bootstrap.pypa.io/get-pip.py | sudo python3.9
pip3.9 install pyspark
```

### Update \~/.bashrc

```bash
nano ~/.bashrc
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3.9
```

Apply changes:

```bash
source ~/.bashrc
```

### Configure Spark Environment

```bash
cd /opt/spark/conf
cp spark-env.sh.template spark-env.sh
nano spark-env.sh
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_MASTER_HOST=192.168.30.95
```

Create logs directory:

```bash
sudo mkdir -p /opt/spark/logs
sudo chown -R $USER:$USER /opt/spark/logs
```

Start Spark Master:

```bash
start-master.sh
jps
```

Should see:

```
10597 Jps
6665 Master
```

### Access Spark Master UI

[http://192.168.30.95:8080](http://192.168.30.95:8080)

---

## Machine B - Worker Setup (`192.168.30.96`)

### Java Installation

```bash
sudo apt install openjdk-17-jdk -y
java --version
```

### Download and Install Spark

```bash
wget https://www.apache.org/dyn/closer.lua/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz
tar -xvf spark-4.0.0-bin-hadoop3.tgz
sudo mv spark-4.0.0-bin-hadoop3 /opt/spark
readlink -f $(which java)
```

### Python and PySpark Installation

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.9 python3.9-distutils -y
curl -sS https://bootstrap.pypa.io/get-pip.py | sudo python3.9
pip3.9 install pyspark
```

### Update \~/.bashrc

```bash
nano ~/.bashrc
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3.9
```

Apply changes:

```bash
source ~/.bashrc
```

### Configure Spark Environment

```bash
cd /opt/spark/conf
cp spark-env.sh.template spark-env.sh
nano spark-env.sh
```

Add:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_WORKER_CORES=2
export SPARK_WORKER_MEMORY=2g
export SPARK_MASTER_URL=spark://192.168.30.95:7077
```

Create logs directory:

```bash
sudo mkdir -p /opt/spark/logs
sudo chown -R $USER:$USER /opt/spark/logs
```

Start Spark Worker:

```bash
start-worker.sh spark://192.168.30.95:7077
jps
```

Should see:

```
10597 Jps
6665 Worker
```

### Access Spark Master UI

[http://192.168.30.95:8080](http://192.168.30.95:8080)

---

## Submitting Applications

### JAR - Client Mode

```bash
spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.30.95:7077 --deploy-mode client /opt/spark/examples/jars/spark-jar.jar 10
```

- Driver runs on Machine A (Master).
- JAR is served over HTTP to workers.

### JAR - Cluster Mode

```bash
spark-submit --class org.apache.spark.examples.SparkPi --master spark://192.168.30.95:7077 --deploy-mode cluster /opt/spark/examples/jars/spark-jar.jar 10
```

- Driver starts on Machine B (Worker).
- JAR must exist on Worker.
- If not present: `java.nio.file.NoSuchFileException` error.

**Solutions:**

- Place JAR on all workers.
- Use shared volume.
- Use HTTP server (e.g., NGINX).
- Use S3/MinIO bucket.

---

## PySpark Files

### Client Mode

```bash
spark-submit --master spark://192.168.30.95:7077 --deploy-mode client /home/syed/pyspark.py
```

- Driver runs on Machine A.

### Cluster Mode

```bash
spark-submit --master spark://192.168.30.95:7077 --deploy-mode cluster /home/syed/pyspark.py
```

- **Not supported** in Spark Standalone for Python apps.
- Error: `Cluster deploy mode is currently not supported for python applications on standalone clusters.`

---

## spark-defaults.conf (`Machine A`)

```
spark.master                     spark://192.168.30.95:7077
spark.driver.memory              1g
spark.driver.cores               1
spark.executor.memory            1g
spark.executor.cores             1
spark.executor.instances         1
```

