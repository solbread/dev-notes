## rpm

#### 주요 명령어

| 명령어 | long 명령어 | 용도              |
| ------ | ----------- | ----------------- |
| -q     | --query     | 패키지 정보 질의  |
| -i     | --install   | 패키지 설치       |
| -U     | --upgrade   | 패키지 업그레이드 |
| -e     | --erase     | 패키지 삭제       |
| -V     | --verify    | 패키지 검증       |
| -K     | --checksig  | 서명 검증         |

### 옵션

- -?, --help : 도움말 출력
- --version : rpm 버전 출력
- **-v : 자세한 정보 출력**
- **-h : print hash marks: 설치 진행 상황을 # 문자를 이용하여 출력**
- -vv : 디버깅용 정보 출력
- --dbpath DIRECTORY_PATH: rpm 데이타베이스 파일 경로 설정. 기본 경로는 /var/lib/rpm 
- --root DIRECTORY_PATH: 파일 시스템의 루트 디렉터리 경로 설정. rpm 을 사용자 디렉터리에 설치했을 경우등에 유용함. 기본 경로는 /
- --pipe CMD: rpm 명령어의 출력을 CMD 명령어로 전송



#### 설치

```rpm -ivh gzip-1.3.12-19.el6_4.x86_64.rpm```

```rpm -ivh http://mirror.centos.org/centos/6/os/x86_64/Packages/gzip-1.3.12-19.el6_4.x86_64.rpm```



#### 업그레이드

```rpm -Uvh my-package.rpm```



#### Ubuntu 데비안 계열

rpm 패키지를 사용할 수 없으므로 .rpm 파일을 .deb 파일로 변환하여 사용해야 한다.

```
#error message
rpm: RPM should not be used directly install RPM packages, use Alien instead!
rpm: However assuming you know what you are doing...
```

[Alien.md](./Alien.md)



#### 참고자료

https://www.lesstif.com/pages/viewpage.action?pageId=7635004