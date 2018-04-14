## Install Java

#### Ubuntu Java 환경변수 설정

```
sudo vi /etc/profile

export JAVA_HOME=/usr/local/jdk_xxx
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=.$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
```



#### Symbolic link 설정

```
cd /usr/java
ln -s java_path latest
ln -s /usr/java/latest default

cd /usr/bin
ln -s /usr/java/default/bin/java java
ln -s /usr/java/default/bin/javac javac
ln -s /usr/java/default/bin/javadoc javadoc
ln -s /usr/java/default/bin/javaws javaws
ln -s /usr/java/default/bin/jps jps
```