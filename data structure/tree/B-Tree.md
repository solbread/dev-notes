## B-Tree

#### 정의

이진트리를 확장하여 하나의 노드가 여러개의 데이터와 자식노드를 가지는 자료구조

한 노드에 최대 M-1개의 데이터가 배치될 수있으면 M차 B트리라고 하며, M이 짝수인지 홀수인지에 따라 알고리즘이 상당히 달라진다. (홀수  B트리가 짝수 B트리에 비해 알고리즘이 훨씬 쉽다.)

* M(Order) : 최대 자식노드의 개수
* key : 노드 내의 데이터를 지칭하는 용어
* overflow : 노드에 B-Tree에서 허용하는 최대 key 개수 보다 많은 key가 저장되어 있는 경우 
* underflow : 노드에 B-Tree에서 허용하는 최소 key 개수 보다 적은 key가 저장되어 있는 경우 



#### 장점

* 더 많은 수의 자식을 가질 수 있게되므로, 같은 개수의 자료도 하나의 레벨에 더 많이 저장되므로 전체적으로 트리의 높이가 낮아짐


* 하나의 노드에 많은 데이터를 가질 수 있음

  * 대량의 데이터를 처리해야 하는 검색 구조에 유용

    > 대량의 데이터는 메모리 보다는 하드디스크 혹은 SSD에 저장되어야 하는데,  이들 외부 기억 장치들은 블럭 단위로 입출력을 하게 되므로 유용하다.
    >
    > 예를 들어 한 블럭이 1024 바이트라면,  2 바이트를 읽으나 1024 바이트를 읽으나 입출력에 대한 비용은 동일하다.  그렇다면 하나의 노드를 1024바이트가 되도록 조절한다면 입출력 면에서 매우 효율적인 구성이 된다.  게다가 균형까지 맞는다면 더욱 더 효율적이다.

    ​

이런 장점이 있기 때문에 실제 B-트리는 많은 데이터베이스 시스템의 인덱스 저장 방법과 파일시스템에 널리 사용되고 있다.



#### 간단한 규칙

* 노드의 key수가 N이면 자식의 수는 항상 N+1이여야 한다. (즉, root노드가 leaf노드인 경우를 제외하고는 항상 최소 2개의 자식노드를 가진다.)
* 노드내의 key는 반드시 (오름차순으로) 정렬된 상태여야 한다.
* 노드의 key의 왼쪽 서브 트리는 해당 key보다 작은 값들로 이루어져 있어야 하고, 오른쪽 서브트리는 해당 key보다 큰 값들로 이루어져 있어야 한다. 이진 검색 트리의 성질과 유사하다.



#### 복잡한 규칙

* root 노드는 자식이 있다면 적어도 2개 이상의 자식노드를 가져야 한다.
* root 노드와 leaf 노드를 제외한 모든 노드는 최소 ┍M/2┑, 최대 M개의 자식노드를 가진다. (M : 차수)
* root 노드와 leaf 노드를 제외한 모든 노드는 적어도 ┍M/2┑-1개의 데이터를 가지고 있어야 한다. (예를 들어 5차 B트리라면 적어도 2개의 데이터를 가지고 있어야 한다.) (M : 차수)
* leaf 노드로 가는 경로의 길이는 모두 같다. 즉, leaf 노드는 모두 같은 레벨에 존재한다.
* 새로운 key 값은 leaf node에 삽입된다.
* 입력 자료는 중복될 수 없다.



#### 3차 B-Tree

![b-tree(2order)](..\picture\b-tree(3order).png)



#### 5차 B-Tree

![b-tree(5order)](..\picture\b-tree(5order).png)



#### 구조

* B-트리의 노드는 대개 항목과 자식 포인터의 순서 집합으로 표현
  * 루트 노드를 제외한 모든 노드는 임의의 수 L, U(L<U)에 대해 최소 L 항목, 최대 U 항목을 가지고 있으며, 최대 U+1개의 자식 포인터를 가지고 있음
  * 모든 내부 노드에서, 자식 포인터의 수는 언제나 항목 개수보다 하나가 많음
  * 모든 리프 노드가 동일한 높이에 있기 때문에, 노드는 일반적으로 리프 노드인지 내부 노드인지 판별하는 수단을 가질 이유가 없음
* 각 내부 노드의 항목은 부트리를 나누는 구분 값
  * 예를 들어, 어떤 내부 노드가 3개의 자식 노드(혹은 부트리)를 가지고 있다면, 그 내부 노드는 두개의 구분 값이나 항목 a1과 a2를 가지고 있어야 한다. 가장 왼쪽 부트리에 있는 모든 값은 a1보다 작으며, 가운데 부트리의 모든 값은 a1와 a2의 사이에 있으며, 가장 오른쪽 부트리의 모든 값은 a2보다 크게 됨



#### Search

탐색은 일반적인 방식, 즉 이진 탐색 트리와 동일한 방식으로 수행된다. 루트에서 시작하여, 하향식으로 탐색 대상의 값을 구분 값과 비교하며 자식 포인터를 찾아나가는 과정으로 진행한다.



#### Insert

* 추가하려는 노드에 key가 들어갈 공간이 있다면?
  * 해당 공간에 데이터를 넣는다.
* 추가하려는 노드에 key가 들어갈 공간이 없다면?
  * 중간값(M이 홀수인 경우에는 M/2, M이 짝수일 경우에는 M/2-1번째)를 선택하여 부모노드로 올린다.



#### Remove

삭제를 하기 위해서는 해당 key를 leaf node로 이동시켜야 한다.

* internal node의 key를 삭제하는 경우

  * key를 자식 노드로 보낸 후 삭제한 후, left 자식노드이면 최대값을, right 자식노드이면 최소값을 부모노드로 올린다.
  * key는 ┍M/2┑개 이상인 자식노드로 보내며,
    1. left 자식노드와 right 자식노드가 둘 다 만족하면, left 자식노드를 선택
    2. left 자식노드와 right 자식노드 중 하나만 만족하면, 만족하는 자식노드를 선택
    3. left 자식노드와 right 자식노드가 둘 다 만족하지 않으면, 두 자식노드를 merge

* leaf node의 key를 삭제하는 경우

  * leaf node의 key가 ┍M/2┑개 이상
    1. key를 삭제
    2. parent node와 sibling node의 key 수가 모두 ┍M/2┑-1개이면 sibling node중 1개와 parent node를 merge
  * leaf node의 키가 ┍M/2┑개 미만 (┍M/2┑-1개)
    1. key를 삭제
    2. parent node에서 leaf node로 보내고, leaf node의 이웃node를 parent node로 보냄
    3. 이 때, 이웃node도 ┍M/2┑-1개 일 경우 underflow가 발생하므로, parent node의 key를 가져와서 leaf node와 이웃node를 merge

* Example

  ![remove b-tree](C:\Users\jungsol\Documents\GitHub\dev-notes\data structure\picture\remove b-tree.PNG)



#### B-Tree 구현

[hochulshin.com DataStructure - B-tree](http://hochulshin.com/data-structure-b-tree/)

[Digital Dynamics - B-트리(B-Tree) 검색](http://ddmix.blogspot.kr/2015/01/cppalgo-18-b-tree-search.html)

[scanfree - B-Tree](http://scanftree.com/Data_Structure/B-Tree)



#### 참고자료

[위키백과 : B트리](https://ko.wikipedia.org/wiki/B_%ED%8A%B8%EB%A6%AC)

[hochulshin.com DataStructure - B-tree](http://hochulshin.com/data-structure-b-tree/)

[scanfree - B-Tree](http://scanftree.com/Data_Structure/B-Tree)

[Digital Dynamics - B-트리(B-Tree) 검색](http://ddmix.blogspot.kr/2015/01/cppalgo-18-b-tree-search.html)

[백인감자 - B-Tree](http://potatoggg.tistory.com/174#recentComments)



#### 기타 자료

[Web Tool : Create B-Tree](https://www.cs.usfca.edu/~galles/visualization/BTree.html)