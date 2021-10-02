## 5. Wordpress 설정
### 1) Wordpress 활성화하기
- 첫번째 EC2 웹서버 터미널로 이동
#### DB 유저, 호스트 정보 입력하기
- Apache 기본 폴더로 이동해서 목록 확인
```
cd /var/www/html
ll
```
- `wp-config-sample.php` 파일 확인 (wp-config는 Wordpress 설정 파일)
- 해당 파일을 복사, 편집하기
```
sudo cp wp-config-sample.php wp-config.php
sudo vi wp-config.php
```
- 다음과 같이 편집(i) ➡ Esc 누르고 `:wq!`로 편집모드 종료
```
define ('DB_NAME', '데이터베이스 이름');
define ('DB_USER', 'DB 유저 아이디')
define ('DB_PASSWORD', 'DB 패스워드);
define ('DB_HOST', 'RDS 엔드포인트');
```
#### Apache가 EC2 내에 폴더를 만들 권한 부여하기
- Wordpress에서 설치할 일부 테마, 응용 프로그램 ➡ EC2 서버 내에 폴더를 만들고 데이터를 저장시킴
- Wordpress가 동작하고 있는 Apache가 EC2 내에 폴더를 자유롭게 만들도록 권한을 부여해야함
`sudo chown -R apache:apache /var/www/html`
#### 복제한 웹서버에도 적용하기
- DB 유저, 호스트 정보 입력
- Apache가 EC2 내에 폴더를 만들 권한 부여하기
- 웹서버가 많으면 많을 수록 반복 작업을 해야함 ➡ 반복작업 없이 한번의 배포로 동시에 서버에 반영되는 툴이 있음 (예 : 쿠버네티스)

### 2) Wordpress 로그인, 테마 적용하기
- `http://(EC2 퍼블릭 주소)/wp-admin/install.php` 이동
- 한국어 선택
- Wordpress <b>관리자 계정 생성</b>
- 로그인 후 좌측 메뉴 외모 > 테마
- `새로 추가` 선택 ➡ oceanwp 선택 (평점 좋고 유명한 테마임)
- 테마 설치 완료 ➡ 활성화 ➡ `사용자 정의하기` 버튼 클릭
- 메뉴에 있는 기능들을 이용하여 쉽게 웹사이트 구축이 가능함

### 3) Wordpress 사이트 접속하기
- ELB 퍼블릭 DNS로 접근
- 관리자 화면은 URL 뒤에 `/wp-admin`을 입력해서 접근 가능

### 4) EC2에서 MySQL 접속하기
- EC2에서 MySQL에 접속하여 DB 데이터 확인 및 편집 가능
- `mysql -E` 옵션은 쿼리 결과를 세로모드로 깔끔하게 보여줌
```
mysql -E -u admin -p -h (RDS 엔드포인트)
```
- Wordpress DB 접속하기
```sql
use wordpress
```
- DB에 생성된 테이블 목록 확인하기
```sql
show tables;
```
- table_name 테이블에 있는 5건의 데이터 확인
```sql
select * from table_name limit 5;
```
