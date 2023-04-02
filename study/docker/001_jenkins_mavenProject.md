## docker Jenkins 에 maven Web 프로젝트 빌드해보기

### 1) Dashboard -> Jenkins 관리 -> Plugins 
- maven 검색 -> Maven Integration 설치

### 2) Dashboard -> 새로운 Item
- Maven Project 선택
- git 정보 입력
![img.png](../_image/docker001_img.png)

- maven 정보 입력
![img_1.png](../_image/docker001_img_1.png)

### 3) 빌드 수행
![img_2.png](../_image/docker001_img_2.png)

docker container 접속해서 아래 명령어 입력시 hello-world.war 보임
![img_3.png](../_image/docker001_img_3.png)
