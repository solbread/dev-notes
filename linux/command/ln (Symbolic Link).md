## ln (Symbolic Link)

#### create symbolic link

```ln -s origin_path symbolic_link_path```



#### remove symbolic link

* 원본은 냅두고 생성된 파일에 대한 심볼릭 링크만 삭제

  ```ln -s origin_path symbolic_link_path```

* 원본은 냅두고 폴더에 대한 심볼릭 링크만 삭제

  ```ln -s origin_path symbolic_link_path``` 	```rm -f symbolic_link_path```

  > rm -f symbolic_link_path를 할 때 폴더 뒤에 /를 붙이지 않는다 ex) rm -f test/link



#### remove origin

원본을 삭제해도 심볼릭링크는 유지되지만, 원본이 없기 때문에 내용은 볼 수 없다.