## 2021 여름 부트캠프 : AWS를 이용한 2-tier 웹 아키텍처 구현
성남산업진흥원 주관 ICT 기업 인턴 교육 프로그램 과제 수행

### 프로젝트 개요
- Amazon Web Service(AWS)를 이용한 2-tier 웹 아키텍처 구현
- Public 서브넷의 웹서버(EC2)와 Private 서브넷의 DB(RDS) 연동
- Wordpress를 기반으로 웹페이지 제작
- 원본 웹서버와 복제본을 ELB(ALB)로 Load Balancing
- Cloudwatch로 RDS 모니터링
<div align="center">
  <img width="600" src="https://user-images.githubusercontent.com/75527311/134180369-1b08be6e-8c32-4ed0-8f73-dd629485c3fe.png">
</div>
<br>

### 구현 방법 요약
- VPC 생성 후 퍼블릭 서브넷에 EC2 인스턴스 생성 ➡ Apache 서버, MySQL 클라이언트, php, Wordpress 설치
- RDS 전용 프라이빗 서브넷 생성 및 RDS 설정
- Wordpress 활성화 및 테마 적용
- EC2 백업 이미지를 생성하여 복제 ➡ ELB로 두 EC2를 연결하여 로드밸런싱
- Cloudwatch에 대시보드에 RDS 지표별 그래프 추가
<br>

### 웹페이지 화면
<div align="center">
  <img src="https://user-images.githubusercontent.com/75527311/134179216-f7439001-9a74-44ea-9692-de278a3cb2aa.PNG">
  <p>자기소개 프로필 페이지</p>
  <br>

  <img src="https://user-images.githubusercontent.com/75527311/134179271-0effd9a0-0b3f-4402-a386-3b4550be9c22.PNG">
  <p>연도별 공부한 기록을 나타낸 페이지 (전공 과목, 자격증, 교육 수료 등)</p>
  <br>

  <img src="https://user-images.githubusercontent.com/75527311/134179565-98285eff-f42a-4d89-9782-cfc3a9e42031.png">
  <p>지금까지 진행한 대표 프로젝트 목록</p>
</div>
