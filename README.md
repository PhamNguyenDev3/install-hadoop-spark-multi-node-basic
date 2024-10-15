### **Hướng Dẫn Cài Đặt Hadoop và Spark Trên Ubuntu**

Dưới đây là các bước chi tiết để cài đặt và cấu hình **Hadoop** và **Spark** trên hệ điều hành **Ubuntu**. Lưu ý: Cài đặt **Hadoop** xong mới tiến hành cài đặt **Spark** để đảm bảo môi trường hoạt động đúng cách.

---

### **Phần 1: Cài Đặt và Cấu Hình Hadoop**

#### **Bước 1: Tạo User Hadoop**

Tạo một user mới tên là `hadoop` và cấp quyền sudo cho user này:

```bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
```

#### **Bước 2: Chuyển sang User Hadoop**

Chuyển sang user `hadoop`:

```bash
su - hadoop
```

#### **Bước 3: Cài đặt JDK 11, Scala và Git**

Cài đặt Java Development Kit (JDK 11), Scala, Git và Neovim:

```bash
sudo apt install openjdk-11-jdk scala git neovim -y
```

#### **Bước 4: Cấu hình IP và Tên Host**

-   Lấy địa chỉ IP của máy:
    ```bash
    ip a
    ```
-   Mở file `/etc/hosts` để gán IP với tên `hadoop`:
    ```bash
    sudo nano /etc/hosts
    ```
-   Thêm dòng sau vào (thay `192.168.x.x` bằng IP của máy):
    ```
    192.168.x.x hadoop
    ```

#### **Bước 5: Tải và Cài Đặt Hadoop**

-   Di chuyển về thư mục gốc của user `hadoop`:
    ```bash
    cd ~
    ```
-   Tải phiên bản Hadoop:
    ```bash
    wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
    ```
-   Giải nén và di chuyển vào thư mục Hadoop:
    ```bash
    tar xzf hadoop-3.4.0.tar.gz
    sudo mv hadoop-3.4.0 ~/hadoop
    ```

#### **Bước 6: Lấy Đường Dẫn JDK**

Sử dụng một trong các lệnh sau để lấy đường dẫn Java:

```bash
readlink -f /usr/bin/javac
# Hoặc
dirname $(dirname $(readlink -f $(which java)))
```

#### **Bước 7: Cấu hình Biến Môi Trường**

-   Mở file `.bashrc` để cấu hình biến môi trường:
    ```bash
    nano ~/.bashrc
    ```
-   Thêm các dòng sau vào cuối file:
    ```bash
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    export HADOOP_HOME=/home/hadoop/hadoop
    export HADOOP_INSTALL=$HADOOP_HOME
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME
    export HADOOP_HDFS_HOME=$HADOOP_HOME
    export HADOOP_YARN_HOME=$HADOOP_HOME
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
    ```
-   Áp dụng thay đổi:
    ```bash
    source ~/.bashrc
    ```

#### **Bước 8: Cấu Hình Hadoop**

##### **8.1 Cấu hình `hadoop-env.sh`**

-   Mở file `hadoop-env.sh` để cấu hình Java:
    ```bash
    nano ~/hadoop/etc/hadoop/hadoop-env.sh
    ```
-   Thêm dòng sau vào:
    ```bash
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    ```

##### **8.2 Cấu hình `core-site.xml`**

-   Mở file `core-site.xml` để cấu hình HDFS:
    ```bash
    nano ~/hadoop/etc/hadoop/core-site.xml
    ```
-   Thêm nội dung sau vào:
    ```xml
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop:9000</value>
        </property>
    </configuration>
    ```

##### **8.3 Cấu hình `hdfs-site.xml`**

-   Mở file `hdfs-site.xml`:
    ```bash
    nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
    ```
-   Thêm nội dung sau:
    ```xml
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
        <property>
            <name>dfs.name.dir</name>
            <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
        </property>
        <property>
            <name>dfs.data.dir</name>
            <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
        </property>
    </configuration>
    ```

##### **8.4 Tạo Thư Mục cho Namenode và Datanode**

-   Tạo thư mục cho `Namenode` và `Datanode`:
    ```bash
    mkdir -p ~/hadoopdata/hdfs/namenode
    mkdir -p ~/hadoopdata/hdfs/datanode
    ```
-   Thay đổi quyền sở hữu và quyền truy cập:

    ```bash
    sudo chown hadoop:hadoop -R ~/hadoopdata/hdfs/namenode
    chmod 755 ~/hadoopdata/hdfs/namenode

    sudo chown hadoop:hadoop -R ~/hadoopdata/hdfs/datanode
    chmod 755 ~/hadoopdata/hdfs/datanode
    ```

#### **Bước 9: Cấu Hình YARN**

-   Mở file `yarn-site.xml`:
    ```bash
    nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
    ```
-   Thêm nội dung sau:
    ```xml
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop</value>
    </property>
    ```

#### **Bước 10: Tạo Key SSH**

-   Tạo key SSH để tránh nhập mật khẩu khi kết nối giữa các node:
    ```bash
    ssh-keygen -t rsa -P ""
    cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
    chmod 0600 ~/.ssh/authorized_keys
    ```

#### **Bước 11: Khởi Động Hadoop**

##### **Khởi động Hadoop**

-   Format Namenode:
    ```bash
    hdfs namenode -format
    ```
-   Khởi động HDFS:
    ```bash
    $HADOOP_HOME/sbin/start-dfs.sh
    ```
-   Kiểm tra trạng thái các dịch vụ:
    ```bash
    jps
    ```
-   Khởi động YARN:
    ```bash
    $HADOOP_HOME/sbin/start-yarn.sh
    ```
-   Kiểm tra trạng thái hệ thống qua URL: [http://hadoop:9870](http://hadoop:9870)

---

### **Phần 2: Cài Đặt và Cấu Hình Spark**

#### **Bước 1: Cài Đặt Spark**

-   Di chuyển về thư mục home của user:
    ```bash
    cd ~
    ```
-   Tải Spark:
    ```bash
    wget https://dlcdn.apache.org/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz
    ```
-   Giải nén và di chuyển Spark:
    ```bash
    tar xvf spark-*.tgz
    sudo mv spark-3.5.3-bin-hadoop3 /opt/spark
    ```
-   Kiểm tra phiên bản Spark:
    ```bash
    /opt/spark/bin/spark-shell --version
    ```

#### **Bước 2: Cấu Hình Spark**

-   Mở file `.bashrc` để cấu hình biến môi trường cho Spark:
    ```bash
    nano ~/.bashrc
    ```
-   Thêm các dòng sau:
    ```bash
    export SPARK_HOME=/opt/spark
    export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
    export PYSPARK_PYTHON=/usr/bin/python3
    ```
-   Áp dụng thay đổi:
    ```bash
    source ~/.bashrc
    ```

#### **Bước 3: Khởi Động Spark**

-   Khởi động Spark Master:
    ```bash
    $SPARK_HOME/sbin/start-master.sh
    ```
-   Truy cập giao diện quản lý tại: [http://127.0.0.1:8080](http://127.0.0.1:8080)

---

Dưới đây là phiên bản viết lại của phần hướng dẫn bạn đã cung cấp về việc clone một máy ảo từ master và cấu hình trên máy slave:

---

### **Clone một Máy Ảo Khác Từ Master: Cấu Hình trên Máy Slave (Nếu Có)**

#### **Bước 1: Đổi Tên Hostname của Slave**

-   Thay đổi hostname cho máy slave:
    ```bash
    sudo hostnamectl set-hostname datanode1
    ```

#### **Bước 2: Cấu Hình Địa Chỉ IP và Tên Host trên Slave**

-   Mở file `/etc/hosts` trên máy slave:
    ```bash
    sudo nano /etc/hosts
    ```
-   Thêm các dòng sau vào file:
    ```
    192.168.x.x hadoop
    192.168.x.x datanode1
    ```

### **Quay Trở Lại Máy Master**

#### **Bước 3: Kết Nối Master và Slave**

-   Kiểm tra kết nối từ master đến `datanode1` bằng lệnh ping:
    ```bash
    ping datanode1
    ```
-   Sao chép khóa SSH từ master sang slave:
    ```bash
    ssh-copy-id hadoop@datanode1
    ```

#### **Bước 4: Cài Đặt Hadoop trên Slave**

-   Mở file `workers` trên master để thêm slave node:
    ```bash
    nano ~/hadoop/etc/hadoop/workers
    ```
-   Thêm `datanode1` vào danh sách:

    ```
    datanode1
    ```

-   Sao chép cấu hình Hadoop từ master sang slave:
    ```bash
    scp $HADOOP_HOME/etc/hadoop/* hadoop@datanode1:$HADOOP_HOME/etc/hadoop/
    ```

### **Bước 12: Khởi Động Hadoop**

#### **Khởi Động Hadoop**

-   Định dạng Namenode:
    ```bash
    hdfs namenode -format
    ```
-   Khởi động HDFS:
    ```bash
    $HADOOP_HOME/sbin/start-dfs.sh
    ```
-   Kiểm tra trạng thái các dịch vụ:
    ```bash
    jps
    ```
-   Khởi động YARN:
    ```bash
    $HADOOP_HOME/sbin/start-yarn.sh
    ```
-   Kiểm tra trạng thái hệ thống qua URL: [http://hadoop:9870](http://hadoop:9870). Nếu không truy cập được datanode với máy slave, hãy thực hiện các lệnh sau trên cả hai máy master và datanode:
    ```bash
    sudo rm -Rf ~/hadoopdata/hdfs/datanode/*
    hdfs namenode -format
    $HADOOP_HOME/sbin/start-all.sh
    ```

### **Cài Đặt Spark**

-   Mở file cấu hình `slaves` của Spark:
    ```bash
    sudo nano /opt/spark/conf/slaves
    ```
-   Thêm `datanode1` vào danh sách.

-   Cấu hình địa chỉ IP của master cho Spark:
    ```bash
    sudo nano /opt/spark/conf/spark-env.sh
    ```
-   Thêm các dòng sau:

    ```bash
    export SPARK_MASTER_HOST=hadoop
    export JAVA_HOME='/usr/lib/jvm/java-11-openjdk-amd64'
    export SPARK_MASTER_PORT=7077
    ```

-   Nếu sử dụng Spark trên YARN, thêm dòng sau:
    ```bash
    export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
    ```

### **4. Khởi Động Cụm Spark**

1. Trên **node master**, khởi động Spark Master:

    ```bash
    $SPARK_HOME/sbin/start-master.sh
    ```

2. Trên **node master**, khởi động các worker:

    ```bash
    $SPARK_HOME/sbin/start-slaves.sh
    ```

    Hoặc để khởi động worker trên từng node, bạn có thể chạy lệnh sau trên mỗi worker:

    ```bash
    $SPARK_HOME/sbin/start-slave.sh spark://hadoop-master:7077
    ```

### **5. Kiểm Tra Cụm Spark**

-   Mở trình duyệt và truy cập vào **Web UI** của Spark Master tại địa chỉ: `http://hadoop-master:8080` để kiểm tra trạng thái của cụm và xác nhận các worker đã kết nối với master hay chưa.
-   Chạy một ứng dụng Spark trên cụm để kiểm tra:
    ```bash
    $SPARK_HOME/bin/spark-shell --master spark://hadoop-master:7077
    ```

---
