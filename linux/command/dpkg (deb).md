## dpkg (deb)

#### deb 파일 설치

```dpkg -i xxx.deb```

```dpkg xxx.deb```



#### 설치된 패키지 목록 조회

```dpkg -l ```

* ii : installed ok installed
* rc : removed 



#### 설치된 패키지에 포함된 파일 보기

```dpkg -L xxx.deb```



#### deb파일의 정보 보기

```dpkg -l xxx.deb```



#### deb 파일 삭제

```dpkg -r xxx.deb```



#### 파일의 패키지명 알아내기

```dpkg -S file_name```



#### 패키지 제거

```dpkg -P package_name```



#### deb파일 포함된 파일들 보기

```dpkg -c package_name```