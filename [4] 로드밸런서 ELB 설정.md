## 4. 로드밸런서 ELB 설정
### 1) ELB의 필요성
- 무거운 프로그램을 처리하려면 스펙 좋은 1대로 처리하는 것이 유리함
- 하지만 웹서버 자체에 무거운 로직이 있는 경우가 드물기 때문에 보통 작은 스펙 서버 2대를 쓰는 경우가 많음

▶ EC2에서 RDB로 정상 접속 가능한지 확인한 후, 복제해서 2대로 만들어볼 것

### 2) EC2 MySQL 접속 테스트
- RDS - 데이터베이스 들어가서 ```연결&보안```에 있는 엔드포인트 복사 (엔드포인트 = DB 접속 주소)
- PuTTy로 EC2 들어가서 MySQL 접속 명령어 입력
```mysql -u admin -p -h (복사한 엔드포인트)```
- `-u` 다음엔 아이디 작성
- `-p` 다음엔 포트 작성 (기본 포트인 3306을 사용하므로 현재는 -p만 작성해도됨)
- -h 다음엔 호스트 주소 (=엔드포인트) 작성
- 접속 성공하면 다음과 같이 나옴
![mysql 접속 성공 PNG](https://user-images.githubusercontent.com/75527311/134528996-9f92d6ce-c16e-47c7-9982-c31c47fecfbe.png)
- <b>using password</b> 오류가 나면 DB 접속은 가능하나, 비밀번호가 틀렸을 것
- <b>connection</b> 오류가 나면 방화벽 설정에 문제가 있는 것으로 VPC, 보안 그룹, 서브넷 등을 확인
- `exit` 입력해서 빠져나오기

### 3) EC2 복제하기
- EC2 인스턴스 선택하고 `작업` - `이미지 템플릿` - `이미지 생성` 클릭
- 이미지 이름과 설명을 작성하고 생성 완료
- 이미지 AMI 메뉴에 들어가면 방금 복제한 AMI가 있음, pending ▶ <b>available</b> 상태로 바뀔 때까지 대기
- 이미지를 선택하고 `시작하기` 클릭
#### 단계 3 : 인스턴스 세부 정보 구성
- **VPC, 퍼블릭 서브넷**을 선택하고 **퍼블릭 IP 자동 할당** 활성화
#### 단계 6 : 보안 그룹 구성
- 기존에 사용하던 웹서버 보안그룹
- 생성 완료 후 관리 목적으로 이름 지어주기

### 4) 로드밸런서 ELB 생성하기
- EC2 - 로드밸런서 - `Load Balancer 생성` 버튼 클릭
- **ALB** 생성하기 ◀ 예전엔 Classic Load Balancer(CLB)를 사용했는데 현재는 관리와 성능이 더 개선된 Application Load Balancer(ALB)를 쓰는 것이 더 좋다고 함
- NLB는 TCP/UDP에 사용되는 것이므로 HTTP를 사용하는 웹서버에서는 쓰이지 않음

#### Basic Configuration
- 이름 작성하기

#### Network mapping
- VPC 선택
- Public 서브넷이 있는 가용영역을 선택하고 (a존) 웹서버가 위치한 Public 서브넷 선택
- ALB는 두 개 이상의 서브넷을 선택하게 되어있으므로 다른 존에 만든 Private 서브넷도 함께 선택 or 다른 존에 Public 서브넷을 만들어서 선택

#### Security groups
- 웹서버의 보안그룹 선택

#### Listeners and Routing
- ALB가 각 웹서버의 상태를 확인하는 방법을 정하는 단계
- `Create target group` 클릭하고 대상 그룹 이름 작성
- 2개의 웹서버(기존 웹서버, 복제한 웹서버)를 선택하고 등록
- 생성한 대상 그룹을 선택하고 ALB 생성 완료

### 5) ALB 작동 확인
- 상태 프로비저닝 중 ▶ 활성 될 때까지 대기
- **DNS 이름**을 복사해서 인터넷 주소창에 붙여넣고 접속하기
- EC2 var/www/html 폴더에 wordpress 파일을 넣었다면 wordpress 안내 페이지가 뜰 것, 아직 넣지 않았다면 Apache Test Page가 뜨는 것이 정상
- ELB의 DNS로 접속하면 한번은 1번 서버, 한번은 2번 서버로 번갈아가며 들어가게 됨

#### 정상 작동 테스트 방법
- var/www/html에 있는 wordpress에서 복사한 index.html 이름을 바꿔보기
```bash
sudo mv /var/www/html/index.html /var/www/html/index.html.bak
```
- 첫번째 웹서버에서 테스트 페이지 만들기
```bash
cd /var/www/html
sudo vi html.index
```
- i를 입력해서 편집 모드로 입장 후 다음과 같이 작성
```HTML
<HTML><BODY><P>webserver 1</P></BODY></HTML>
```
- esc 누르고 :wq! 입력해서 편집모드 빠져나오고
- 두번째 웹서버도 마찬가지로 테스트 페이지 만들기
```html
<HTML><BODY><P>webserver 2</P></BODY></HTML>
```
- ELB 퍼블릭 DNS로 접속해서 새로고침을 누르면 한번은 1번 서버, 한번은 2번 서버로 접속됨
- 테스트 후에는 생성했던 index.html을 지우고 wordpress의 원래 파일을 복원시킴
```bash
sudo rm -rf /var/www/html/index.html
sudo mv /var/www/html/index.html.bak /var/www/html/index.html
```

▶ ELB로 부하를 분산시키는 웹서버 아키텍처 완성!😎
