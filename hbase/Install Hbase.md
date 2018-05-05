## Install Hbase

#### Intall Standalone Hbase

1. hbase 다운 및 압축해제 [download stable hbase](http://apache.mirror.cdnetworks.com/hbase/stable/)

2. JAVA_HOME 등록

   - conf/hbase-env.sh 파일에서 `export JAVA_HOME=` 부분 주석해제하고 경로지정

3. hbase 설정파일 변경

   - conf/hbase-site.xml 

     ```xml
     <configuration>
       <property>
         <name>hbase.rootdir</name>
         <value>file:///home/testuser/hbase</value>
       </property>
       <property>
         <name>hbase.zookeeper.property.dataDir</name>
         <value>/home/testuser/zookeeper</value>
       </property>
       <property>
         <name>hbase.unsafe.stream.capability.enforce</name>
         <value>false</value>
         <description>
           Controls whether HBase will check for stream capabilities (hflush/hsync).

           Disable this if you intend to run on LocalFileSystem, denoted by a rootdir
           with the 'file://' scheme, but be mindful of the NOTE below.

           WARNING: Setting this to false blinds you to potential data loss and
           inconsistent system state in the event of process and/or node failures. If
           HBase is complaining of an inability to use hsync or hflush it's most
           likely not a false positive.
         </description>
       </property>
     </configuration>
     ```

4. hbase 실행 : `bin/start-hbase.sh`



#### Refeence 

http://hbase.apache.org/book.html#quickstart

