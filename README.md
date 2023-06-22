≡ Руководство по выполнению тестового задания: Запустить и получить успешное завершение action в планировщике Apache Oozie

# Настройка виртуальной машины

* Скачиваем и устанавливаем ***VirtualBox***
* Скачиваем образ ***Ubuntu Server 22.04.2*** (.iso файл)
* Создаем виртуальную машину(далее ВМ) с характеристиками:
    * ОС - ***Ubuntu Server 22.04.2***
    * Процессор - ***2 ядра***
    * Оперативная память - ***4096 МБ***
    * Размер диска - ***25 ГБ***
    * Имя пользователя - ***hduser***
    * Выбираем ***Install OpenSSH server***
    * В настройках в разделе *Сеть* выбираем *Тип подключения* - ***Сетевой мост*** (для получения IP адреса из локальной сети)
* Вводим логин и пароль нашего пользователя
* Узнаем IP адрес нашей ВМ:
  ```bash
  ip a
  ```
* Переходим в ***WSL***
* Генерируем пару ключей:
  ```bash
  ssh-keygen
  ```
* Копируем публичный и приватный ключ в ВМ (копирование приватного ключа необходимо для правильной работы hadoop):
  ```bash
  ssh-copy-id hduser@192.168.0.7
  scp id_rsa hduser@192.168.0.7:~/.ssh
  ```
* Подключаемся к ВМ:
  ```bash
  ssh hduser@192.168.0.7
  ```
* Проверяем можем ли подключится к локальному хосту по ***ssh***:
  ```bash
  ssh localhost
  ```
* Обновляем пакеты:
  ```bash
  sudo apt update
  sudo apt upgrade
  ```  



# Настройка Hadoop-2.6.0 Single Node Cluster
* В официальной документации на сайте ***Apache*** написано, что необходимо установить ***ssh*** и ***pdsh***. ***Ssh*** было установлено при создании ВМ. Устанавливаем ***pdsh***:
  ```bash
  sudo apt install pdsh
  ``` 
* Также требуется ***java 8*** и ***maven***, устанавливаем:
  ```bash
  sudo apt install openjdk-8-jre-headless openjdk-8-jdk
  sudo apt install maven
  ``` 
* Скачиваем архив ***hadoop-2.6.0*** в домашнюю директорию пользователя:
  ```bash
  wget https://archive.apache.org/dist/hadoop/core/hadoop-2.6.0/hadoop-2.6.0.tar.gz
  ``` 
* Разархивируем скачанный файл:
  ```bash
  sudo tar -xzvf hadoop-2.6.0.tar.gz
  ``` 
* Переименовываем директорию с разархивированным содержимым для удобства:
  ```bash
  mv hadoop-2.6.0 hadoop
  ``` 
* Cоздаем группу ***hadoop***:
  ```bash
  sudo addgroup hadoop
  ``` 
* Добавляем пользователя ***hduser*** в группу ***hadoop***:
  ```bash
  sudo usermod -a -G hadoop hduser
  ``` 
* Открываем файл ***/etc/sudoerc*** и добавляем туда нашего пользователя:
  ```bash
  sudo nano hadoop /etc/sudoers
  ``` 
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/1%202.jpg)
* Делаем нашего пользователя владельцем директории ***hadoop*** и всего ее содержимого:
  ```bash
  sudo chown -R hduser:hadoop hadoop
  ``` 
* Добавляем в файл ***.bashrc*** переменные окружения hadoop:
  ```bash
  nano .bashrc
  ``` 
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/2.jpg)
* Переподключаемся, чтобы подгрузились переменные окружения:
  ```bash
  exit
  ssh hduser@192.168.0.7
  ``` 
* Добавляем конфигурацию в ***core-site.xml***:
  ```bash
  nano hadoop/etc/hadoop/core-site.xml
  ``` 
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/Pasted%20image%2020230622185518.png)
* Добавляем конфигурацию в ***hdfs-site.xml***:
  ```bash
  nano hadoop/etc/hadoop/hdfs-site.xml
  ``` 
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/4.jpg)
* Форматируем файловую систему hadoop:
  ```bash
  hdfs namenode -format
  ``` 
* Запускаем ***NameNode daemon*** и ***DataNode daemon***:
  ```bash
  start-dfs.sh
  ``` 
* Проверяем доступность ***NameNode*** в веб-интерфейсе:
	  http://192.168.0.7:50070/ 
	  ***50070*** - порт по умолчанию для ***NameNode***
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/7.jpg)
* Создаем директории в ***hdfs***, необходимые для выполнения заданий ***MapReduce***:
  ```bash
  hdfs dfs -mkdir /user
  hdfs dfs -mkdir /user/hduser
  ``` 
* Добавляем конфигурацию в ***mapred-site.xml***:
  ```bash
  nano hadoop/etc/hadoop/mapred-site.xml
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/5.jpg)
* Добавляем конфигурацию в ***yarn-site.xml***:
  ```bash
  nano hadoop/etc/hadoop/yarn-site.xml
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/6.jpg)
* Запускаем ***ResourceManager daemon*** и ***NodeManager daemon***:
  ```bash
  start-yarn.sh
  ```
* Проверяем доступность ***ResourceManager*** в веб-интерфейсе:
	  http://192.168.0.7:8088/ 
	  ***8088*** - порт по умолчанию для ***ResourceManager***
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/8.jpg)



# Настройка Oozie-5.2.1
* Скачиваем архив ***Oozie-5.2.1*** в домашнюю директорию пользователя:
  ```bash
  wget https://archive.apache.org/dist/oozie/5.2.1/oozie-5.2.1.tar.gz
  ```
* Разархивируем скачанный файл:
  ```bash
  tar -xzvf oozie-5.2.1.tar.gz
  ```
* Проверяем версию ***hadoop*** в ***pom.xml***:
  ```bash
  nano oozie-5.2.1/pom.xml
  ```
* Собираем ***Oozie*** с помощью скрипта ***mkdistro.sh***
	При сборке обнаружилось, что не удается собрать зависимости org.apache.hive.hcatalog:hive-hcatalog-server-extensions:jar:1.2.2 -> org.apache.hive.hcatalog:hive-hcatalog-core:jar:1.2.2 ->
    org.apache.hive:hive-cli:jar:1.2.2 -> 
    org.apache.hive:hive-service:jar:1.2.2 -> 
    org.apache.hive:hive-exec:jar:1.2.2 -> 
    org.apache.calcite:calcite-core:jar:1.2.0-incubating -> 
    org.pentaho:pentaho-aggdesigner-algorithm:jar:5.1.5-jhyde, 
    т.к. страница сайта http://conjars.org/repo, где хранятся репозитории с зависимостями, недоступна более.
 Поэтому для успешной сборки используем флаг ***--fail-never***:
   ```bash
  bin/mkdistro.sh -DskipTests --fail-never
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/Pasted%20image%2020230622191134.png)
* Создаем директорию ***libext*** по указанному пути и переходим в нее:
   ```bash
  mkdir /home/hduser/oozie-5.2.1/distro/target/oozie-5.2.1-distro/oozie-5.2.1/libext
  cd libext
  ```
* Скачиваем архив, необходимый для запуска ***Oozie Web Console***:
   ```bash
  wget http://archive.cloudera.com/gplextras/misc/ext-2.2.zip
  ```
* Добавляем в файл ***.bashrc*** переменную ***OOZIE_HOME***:
   ```bash
  nano /home/hduser/.bashrc
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/9.jpg)
* Переподключаемся, чтобы подгрузились переменные окружения:
  ```bash
  exit
  ssh hduser@192.168.0.7
  ``` 
* Копируем библиотеки ***hadoop*** в директорию ***libext***:
  ```bash
  cp $HADOOP_HOME/share/hadoop/common/*.jar $OOZIE_HOME/libext
  cp $HADOOP_HOME/share/hadoop/common/lib/*.jar $OOZIE_HOME/libext     
  cp $HADOOP_HOME/share/hadoop/hdfs/*.jar $OOZIE_HOME/libext
  cp $HADOOP_HOME/share/hadoop/hdfs/lib/*.jar $OOZIE_HOME/libext
  cp $HADOOP_HOME/share/hadoop/mapreduce/*.jar $OOZIE_HOME/libext
  cp $HADOOP_HOME/share/hadoop/mapreduce/lib/*.jar $OOZIE_HOME/libext
  cp $HADOOP_HOME/share/hadoop/yarn/*.jar $OOZIE_HOME/libext
  cp $HADOOP_HOME/share/hadoop/yarn/lib/*.jar $OOZIE_HOME/libext
  ``` 
* Добавляем конфигурацию в ***oozie-site.xml***:
  ```bash
  nano $OOZIE_HOME/conf/oozie-site.xml
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/Pasted%20image%2020230622185651.png)
* Устанавливаем ***unzip***, необходимый для запуска скрипта ***oozie-setup.sh***:
  ```bash
  sudo apt install unzip
  ```
* Переходим в ***$OOZIE_HOME*** и запускаем команду для настройки ***Oozie***:
  ```bash
  cd $OOZIE_HOME
  bin/oozie-setup.sh sharelib create -fs /user/hduser/share/lib
  ```
* Запускаем ***Oozie***:
  ```bash
  bin/oozied.sh start
  ```
* Проверяем доступность ***Oozie*** в веб-интерфейсе:
	  http://192.168.0.7:11000/oozie 
	  ***11000*** - порт по умолчанию для ***Oozie***:
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/10.jpg)
* Также можно проверить статус работы ***Oozie***:
  ```bash
  ./bin/oozie admin -oozie http://localhost:11000/oozie -status
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/Pasted%20image%2020230622191330.png)



# Запуск action в Oozie
* В директории ***$OOZIE_HOME*** распаковываем архив ***oozie-examples.tar.gz***:
  ```bash
  tar -zxf oozie-examples.tar.gz
  ```
* Копируем директорию с разархивированным содержимым в ***hdfs***:
  ```bash
  hadoop fs -put examples examples
  ```
* Выбираем пример для запуска, в нашем случае ***shell***, переходим в эту директорию  и редактируем файл ***job.properties***, т.к. там указан неверный порт (***9000*** - порт по умолчанию):
  ```bash
  nano $OOZIE_HOME/examples/apps/shell/job.properties
  ```
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/Pasted%20image%2020230622192716.png)
* Запускаем ***Shell Action***:
  ```bash
  bin/oozie job -oozie http://localhost:11000/oozie -config         $OOZIE_HOME/examples/apps/shell/job.properties -run
  ```
* Проверяем выполнение нашего ***Shell Action*** в веб-интерфейсе
	  http://192.168.0.7:11000/ 
	  ***11000*** - порт по умолчанию для ***Oozie***:
![](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/1.jpg)
* Также можно проверить в консоле с помощью команды:
  ```bash
  bin/oozie job -oozie http://localhost:11000/oozie -info 0000000-230622092656254-oozie-hdus-W
  ```
![[](https://github.com/MarinaReic/Hadoop-with-Oozie/blob/master/screenshots/2%201.jpg)
