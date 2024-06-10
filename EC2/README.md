- 루트 유저
    - 생성한 계정의 모든 권한을 자동으로 가지고 있음
    - 생성시 만든 이메일 주소로 로그인
    - 탈취당했을 때 복구가 힘듦 : MFA 설정 필요
    - 루트 유저는 관리용으로만 이용 : 계정 설정 변경, 빌링 등
    - AWS API 호출 불가(Access key/Secret Accesskey 부여불가)

- IAM 유저
    - 만들 때 주어진 아이디로 로그인
    - 기본 권한이 없음 : 따로 권한을 부여해야 함
        - 예 : 관리자 IAM User, 개발자 IAM User 등
    - AWS API 호출 가능
        - AccessKey : 아이디 개념
        - Secret Access Key : 패스워드 개념
    - AWS의 관리를 제외한 모든 작업은 관리용 IAM User를 만들어 사용할 것
    - 권한 부여시 루트유저와 같이 모든 권한을 가질 수 있지만 빌링 관련 권한은 루트 유저가 허용해야 함

# AWS 가상화 개념

단일 컴퓨터의 하드웨어 요소를 일반적으로 가상 머신이라고 하는 다수의 가상 컴퓨터로 분할할 수 있도록 해주는 기술 

# AWS 계정 생성 후 첫 설정

1. 계정 생성 후 MFA 설정
2. IAM - 계정 별칭 - IAM 계정으로 로그인 시 필요하므로 Root 계정 별칭 만들어주기
3. IAM 유저 생성하기
    - 이름 : admin
    - AWS Management Console 액세스 - 사용자 지정 비밀번호 설정 - 비밀번호 재설정 필요 체크해제
    - 직접 정책 연결 - 권한 정책 - AdministratorAccess (빌링을 제외한 모든 AWS 권한을 가짐)
    - csv 다운로드
4. IAM 유저 MFA 설정 (Root 와 동일한 방식으로)
5. Root 로그아웃 후 IAM admin으로 로그인 

# IAM (AWS Identity and Access Management)

액세스 관리와 허용 및 거부

# EC2 한대를 프로비전하고 웹 서버 구성하기

1. EC2를 구성하기 위한 AMI 선택
2. EC2의 유형과 사이즈 선택
3. EBS 설정
4. 보안 그룹 설정 (HTTP, 80 SSH 22)
5. EC2 생성
6. EC2 접속 후 웹 서버 설치 및 웹 서버 실행
7. 웹브라우저에서 접속 테스트

sudo -s : root 권한 가져오기

yum install httpd -y : apache server 다운로드

service httpd start : apache server 실행

Error : GET public IP 403 (Forbidden)

해결 방법 :
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd # 아파치 서비스 시작 
systemctl enable httpd # 시스템이 재부팅하면 자동으로 서비스 실행하도록 등록 
-> Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.

echo " <h1> Hello world from $(hostname -f) </h1> " > /var/www/html/index.html
```

service 와 systemctl 명령어 차이점 : https://heni.tistory.com/22

# EC2의 가격정책

- On-Demand
    - 실행하는 인스턴스에 따라 시간 또는 초당 컴퓨팅 파워로 측정된 가격을 지불
    - 약정은 필요 없음
    - 장기적인 수요 예측이 힘들거나, 유연하게 EC2를 사용하고 싶을 때
    - 한번 써보고 싶을 때
- Spot Instance
    - 경매 형식으로 시장에 남는 인스턴스를 저렴하게 구매해서 쓰는 방식
    - 최대 90%정도 저렴
    - 단 언제 도로 내주어야 할지 모름…..
    - 시작 종료가 자유롭거나 추가적인 컴퓨팅 파워가 필요한 경우
    - 반환 시간 예측 불가능
        - 인스턴스가 확보되고 종료되는 것을 반복해도 문제 없는 분산 아키텍처 필요
    - 주로 빅데이터 처리, ML 등 많은 인스턴스가 필요한 작업에 사
- 예약 인스턴스(Reserved Instance-RI)
    - 미리 일정 기간(1~3년) 약정해서 쓰는 방식
    - 최대 75% 저렴
    - 수요 예측이 확실할 때
    - 총 비용을 절감하기 위해 어느정도 기간의 약정이 가능한 사용자
- 전용 호스트(Dedicated)
    - 실제 물리적인 서버를 임대하는 방식
    - 라이선스 이슈 (Window Server 등)
    - 규정에 따라 필요한 경우
    - 퍼포먼스 이슈 (CPU Streal 등)
    
- 가격 순서
    - 스팟 인스턴스 < 예약 인스턴스 < 온디맨드 < 전용 호스트
- EC2의 가격모델은 EBS와는 별도
- 기타 데이터 통신 등의 비용은 별도로 청구
    - 참고로 AWS는 AWS 바깥으로 나가는 트래픽에 대해서만 요금을 부과
    

# AWS EBS

- 간단히 말하자면 가상 하드 드라이브
- EC2 인스턴스가 종료되어도 계속 유지 가능
- 인스턴스 정지 후 재 기동 가능
- 하나의 EBS를 여러 EC2에 장착 가능
- 루트 볼륨으로 사용시 EC2가 종료되면 같이 삭제됨
    - 단 설정을 통해 EBS만 따로 존속 가능
- EC2와 같은 가용영역이 존재

## Snapshot

- 특정 시간에 EBS 상태의 저장본
    - EBS에 사진을 찍어둔 개념
- 필요 시 스냅샷을 통해 특정 시간의 EBS를 복구 가능
- S3에 보관
    - 증분식 저장 → 초기 스토리지 용량을 설정 후 기존에서 변화된 부분만 따로 떼어 저장/삭제한다.

## AMI

- EC2 인스턴스를 실행하기 위해 필요한 정보를 모은 단위
    - OS, 아키텍쳐 타입(32bit or 64bit), 저장공간 용량
- AMI를 사용하여 EC2를 복제하거나 다른 리전 or 계정으로 전달 가능
- 스냅샷 기반으로 AMI 구성 가능
- 구성
    - 1개 이상의 EBS 스냅샷
    - 사용 권한(어떤 aws account가 사용할 수 있는지)
    - 블록 디바이스 맵핑(EC2 인스턴스를 위한 볼륨 정보 = EBS가 무슨 용량으로 몇개 붙는지)
    - 인스턴스 저장 인스턴스의 경우 루트 볼륨에 대한 템플릿 (ex: 운영체제, 애플리케이션 등)
- 총 두가지 타입
    - EBS 기반
    - 인스턴스 저장 기반
- 타입에 따른 AMI의 생성방법
    - EBS: 스냅샷을 기반으로 루트 디바이스 생성
    - 인스턴스 저장: S3에 저장된 템플릿을 기반으로 생성
