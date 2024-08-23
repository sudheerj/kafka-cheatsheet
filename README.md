# kafka-cheatsheet

## Install Kafka with Zookeeper on Windows
1. Install WSL2 using `wsl --install`

2. Install Java JDK version 11

3. Download Apache Kafka from https://kafka.apache.org/downloads under Binary Downloads section or use wget command

    ```bash
    wget https://archive.apache.org/dist/kafka/3.8.0/kafka_2.13-3.8.0.tgz
    ```
    Add below system run command in `.bashrc`

    ```bash
    PATH="$PATH:~/kafka_2.13-3.8.0/bin"
    ```

4. Extract the contents on WSL2 and setup the $PATH environment variables to easy access the Kafka binaries
   ```bash
   tar xzf kafka_2.13-3.0.0.tgz
   mv kafka_2.13-3.0.0 ~
   ```

5. Start Zookeeper using the binaries in WSL2
   ```bash
   zookeeper-server-start.sh ~/kafka_2.13-3.8.0/config/zookeeper.properties
   ```

6. Start Kafka using the binaries in an another process in WSL2

   ```bash
   kafka-server-start.sh ~/kafka_2.13-3.8.0/config/server.properties
   ```