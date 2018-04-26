## Partition Selection

* Partition Option

  ```sql
  PARTITION (partition_names)

  partition_names:
      partition_name, ...
  ```

  ​

* 특정 Partition의 데이터를 조회

  ```sql
  SELECT * FROM table PARTITION (partition1, partition2, ...)
  ```



출처

https://dev.mysql.com/doc/refman/5.6/en/partitioning-selection.html