## Background Task

#### "&"

* task 실행 명령어 뒤에 &를 붙여서 백그라운드 작업

* 터미널에서 세션을 끊어버리면 task가 종료됨 -> nohup 이용

  > &는 백그라운드로 돌린다는 의미이며, 기본적으로는 nohup이 아닐 경우 터미널 세션이 끊어지면 실행도 끊어진다. 하지만 요즘 옵션에는 nohup과 같은 동작을 하도록 설정이 되어있어서 &만으로도 nohup과 같은 동작을 하게 된다.

* example :  `xxx.sh &`



#### nohup

* 시작
  * `nohup ${task} &`
  * `nohup ${task} > ${log_name} &`
  * `nohup ${task} > ${std_log_name} 2 > ${error_log_name} &`
  * 확인 `ps -ef ${task_name}
* 종료 `kill ${pid}`
* 로그
  * nohup으로 실행된 쉘 스크립트는 자동으로 nohup.out 이라는 이름으로 nohup을 실행한 위치에 자동생성