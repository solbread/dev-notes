## Setting JMX

```shell
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote=true"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.port=1099"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.access.file=Path_to_access_file/jmxremote.access"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.password.file=Path_to_password_file/jmxremote.password"
```



#### JMX url

`service:jmx:rmi:///jndi/rmi://${host}:${port_number}/jmxrmi`