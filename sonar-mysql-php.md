**Prerequisites**

**Install JAVA**

```cmd
sudo apt-get update
sudo apt-get install default-jre
```
** Install MySQL**

```cmd
sudo apt-get update
sudo apt-get install mysql-server
```
**Create SonarQube Database and User**
```sql
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'sonar'@'%' IDENTIFIED BY 'sonar';
GRANT ALL PRIVILEGES ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';
GRANT ALL PRIVILEGES ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
FLUSH PRIVILEGES;
```
**Changes in MySQL Configuration**

```ini
innodb_buffer_pool_size = 2G       #Ram,we should give about above 60%
innodb_log_file_size = 256M
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit = 2
max_connections = 300
query_cache_size = 64M
query_cache_type = 1
wait_timeout = 300
interactive_timeout = 300

#Disable reverse DNS lookups
skip-name-resolve
```
**Restart mysql**

```cmd
sudo systemctl restart mysql
```
**OS System Settings**
* To prevent resource bottlenecks and ensure SonarQube runs smoothly, adjust your system settings by editing the /etc/sysctl.conf file.

```cmd
sudo vim /etc/sysctl.conf
```
* Some of the valuse are important so accarding 

```ini
fs.file-max = 65536

# Virtual Memory Settings
vm.max_map_count = 262144
vm.swappiness = 10

# Network Stack Tuning
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_fin_timeout = 15

# Shared Memory Settings
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
```

```cmd
sudo sysctl -p
```
**Linux Limits Configuration**

```cmd
sudo vim /etc/security/limits.conf
```
```ini
sonar   soft    nofile   65536
sonar   hard    nofile   65536
sonar   soft    nproc    4096
sonar   hard    nproc    8192
```

**Install SonarQube**
```cmd
export SONAR_VERSION=  #add latest -version 
```

** Unzip SonarQube**

```cmd
wget http://dist.sonar.codehaus.org/sonarqube-${SONAR_VERSION}.zip
unzip sonarqube-${SONAR_VERSION}.zip
sudo mv sonarqube-${SONAR_VERSION} /opt/sonar
```
**Configure SonarQube**

* we need to edit sonar.properties file

```cmd
sudo vim /opt/sonar/conf/sonar.properties
```
```ini
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
sonar.web.host=127.0.0.1 # or 0.0.0.0
sonar.web.context=/sonar
sonar.web.port=9000 # chage the port, not defult
```
**Memory Settings**
* To avoid memory issues and improve performance, increase the heap memory allocation by adding these properties (depending on the available system memory)
```ini
sonar.web.javaOpts=-Xms512m -Xmx2048m -XX:+HeapDumpOnOutOfMemoryError
sonar.ce.javaOpts=-Xmx2048m -XX:+HeapDumpOnOutOfMemoryError
sonar.search.javaOpts=-Xms512m -Xmx1024m -XX:+HeapDumpOnOutOfMemoryError
```

* start sonar server or create sonar systemd service 
```cmd
sudo /opt/sonar/bin/linux-x86-64/sonar.sh start
```
**SonarQube Runner Installation**

_Set the SonarQube Runner Version_

```cmd
export SONAR_RUNNER_VERSION= #lates
```
```cmd
wget http://repo1.maven.org/maven2/org/codehaus/sonar/runner/sonar-runner-dist/${SONAR_RUNNER_VERSION}/sonar-runner-dist-${SONAR_RUNNER_VERSION}.zip
unzip sonar-runner-dist-${SONAR_RUNNER_VERSION}.zip
sudo mv sonar-runner-${SONAR_RUNNER_VERSION} /opt/sonar-runner
```
```cmd
sudo vim /opt/sonar-runner/conf/sonar-runner.properties
```
```ini
sonar.host.url=http://localhost:9000/sonar
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
```

**Set Environment Variables for pam**

_Create or Edit `~/.pam_environment`_

```bash
sudo touch ~/.pam_environment
sudo vim ~/.pam_environment
```
_Add the following lines_

```ini
SONAR_RUNNER_HOME=/opt/sonar-runner
PATH DEFAULT=${PATH}:${SONAR_RUNNER_HOME}/bin
```
* Restart the SonarQube Server again.

* Download the PHP plugin and place it in the extensions/plugins directory

* wget the file and add it below location

```cmd
sudo mv downloaded-plugin.jar /opt/sonar/extensions/plugins/
```
* Restart the SonarQube Server again.

**Running SonarQube Analysis on a Project**
* Navigate to your project root directory.
 * Run the SonarQube analysis:

* Navagate to sonar web page and create Project Name and Key

```cmd
sonar-runner -Dsonar.host.url=http://localhost:9000/sonar -Dsonar.projectKey=rcms-org -Dsonar.projectName=rcms-org -Dsonar.projectVersion=1.0 -Dsonar.sources=./app -Dsonar.language=php -Dsonar.jdbc.url=jdbc:mysql://localhost:3306/sonar -Dsonar.jdbc.username=sonar -Dsonar.jdbc.password=sonar
```


