## Index Type : B-Tree Index (Balanced-Tree Index)

#### B-Tree Index (Balanced-Tree Index)

* 가장 일반적으로 사용되고, 가장 먼저 도입되었으나, 아직 가장 범용적인 목적으로 사용되는 인덱스 알고리즘
* B+-Tree 또는 B*-Tree가 사용됨
* B-Tree는 칼럼의 원래 값을 변형시키지 않고 (물론 값의 앞부분만 잘라서 관리하기는 하지만) 인덱스 구조체 내에서는항상 정렬된 상태로 유지하고 있음
* 전문 검색과 같은 특수한 요건이 아닌 경우, 대부분 인덱스는 거의 B-Tree를 사용할 정도로 이반적인 용도에 적합한 알고리즘



#### 구조

![b-tree index](C:\Users\jungsol\Documents\GitHub\dev-notes\mysql\picture\b-tree index.gif)







#### 참고자료

개발자와 DBA를 위한 Real MySQL, 저 이성욱