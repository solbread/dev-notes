## git clone



#### 기존 repository의 모든 파일과 커밋을 그대로 복제하여 새로운 repository를 만드는 방법

1. 기존 repository의 https 주소를 확인

   ```git remote -v```

2. 기존 repository을 bare mirrored clone하여 .git파일 생성

   ```git clone --mirror ${old_repository_https}```

3. clone한 기존 repository로 이동

   ``` cd ${repository-to-mirror.git} ```

4. 새로운 repository에 기존 repository를 연결

   ``` git remote set-url --push origin ${new_repository_https}```

5. push

   ``` git push --mirror ```

출처 https://help.github.com/articles/duplicating-a-repository/