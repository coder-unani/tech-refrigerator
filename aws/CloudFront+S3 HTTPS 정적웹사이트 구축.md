요즘의 웹사이트 개발은 FrontEnd는 HTML,CSS,JavaScript로 구성하고, API로 백엔드를 호출하는 Jemstack 형태로 개발되는 추세입니다.
JAMstack은 JavaScript,API,Markup으로 사이트를 구성하고 클라이언트의 배포는 CDN으로 이루어지는 형태입니다. (https://jamstack.org/)

AWS의 CloudFront서비스와 S3서비스를 결합하여 이러한 인프라를 구축하는 방법을 알아보겠습니다.

1. 도메인 등록
AWS Route 53에서 도메인을 등록하여도 되지만 co.kr, kr 등의 일부 국가의 도메인은 지원을 하지 않습니다.
저는 kr과 co.kr 도메인을 사용할 예정이어서 국내 도메인 업체에서 등록하였습니다.
국내 등록업체로는 whois, cafe24, gabia 등이 있습니다.
![](https://velog.velcdn.com/images/unani/post/32d0a4e8-96cf-4e53-b728-79f3c3f0d9d1/image.png)
도메인을 등록하셨으면 AWS Console로 와서 Route 53에서 호스팅영역을 생성합니다.
AWS Console > Route 53 > 호스팅 영역 > 호스팅 영역 > 호스팅 영역 생성 버튼 순으로 클릭하면 됩니다.
호스팅 생성 메뉴에서 기존에 등록한 도메인을 입력합니다.
![](https://velog.velcdn.com/images/unani/post/db28cb00-0b9b-4f60-b648-0b612c2416b6/image.png)
호스팅 영역 등록 후 확인되는 AWS DNS를 기억해둡시다.
![](https://velog.velcdn.com/images/unani/post/f6e198bd-21c6-4297-8072-60787ce0f885/image.png)
기존에 도메인을 등록한 사이트(ex. whois,cafe24, gabia 등)에 가서 네임서버 변경 서비스를 통해 적어둔 AWS DNS로 변경합니다
![](https://velog.velcdn.com/images/unani/post/4ab91369-ef91-46fb-82b1-24c1c1103109/image.png)
DNS에 등록된 도메인은 최초등록 및 변경사항이 발생할 때 마다 전파되는 시간이 필요합니다. 짧게는 수분에서 수시간이 소요될 수 있습니다.
아래 명령어를 사용하면 도메인이 DNS를 잘 바라보고 있는지 확인 할 수 있습니다.
![](https://velog.velcdn.com/images/unani/post/04643bac-0362-43b1-a8ca-5d2fb60dcfcc/image.png)

2. SSL Certification(인증서) 발급받기
인증서를 발급받기 위해서는 이 도메인을 사용자가 완벽하게 제어하고 있다는 것을 인증해야 합니다.
보통 버킷이나 EC2 인스턴스가 생성된 리전에서 인증서를 발급받게 되는데 cloudFront의 경우는 미국 동부(버지니아 북부) 리전(us-east-1)에서
인증서를 발급받아야 합니다.
AWS Console > Certificate Manager로 들어가서 오른쪽 상단에 위치한 리전을 버지니아 북부로 설정합니다.
인증서 요청 > 퍼블릭 인증서 요청을 클릭하고 다음 버튼을 누릅니다.
도메인 이름에 cloudFront에서 사용할 도메인을 입력하고 "이 인증서에서 다른 이름 추가"도 필요하다면 등록해줍니다.
![](https://velog.velcdn.com/images/unani/post/f61efd32-97e1-4d2f-9813-900d9305f2c1/image.png)
AWS 권장사항인 DNS를 인증을 선택했습니다. DNS 인증과정에서 막히신 분들은 이메일 인증을 선택하셔도 됩니다.
이메일 인증은 whois 도메인 검색 사이트에서 해당 도메인을 검색했을때 나오는 소유주의 이메일로 인증메일이 발송됩니다.
저는 이미 인증성공해서 상태값이 성공이지만 처음 등록하면 검증 대기 중 상태가 됩니다.
![](https://velog.velcdn.com/images/unani/post/e93d445e-da44-4014-a213-6c86c2227811/image.png)
검증 대기 중 상태에서 Route 53에서 레코드 생성 버튼을 누르면 아래와 같은 확인창이 나오게 됩니다.
레코드 생성 버튼을 누르고 검증이 완료될 때까지 대기합니다.
직접 CNAME을 입력해도 되지만 DNS에 따라서 CNAME 끝에 .을 붙여야 하는 경우도 빼야 하는 경우도 있어서 되도록이면
Route 53에서 레코드 생성 버튼을 눌러서 진행하길 추천합니다.
![](https://velog.velcdn.com/images/unani/post/f6b7b610-2490-4b1d-8341-849f112241ce/image.png)
경험상 보통 5분에서 10분에 검증이 완료되지만 장시간 검증 대기 중 상태라면 설정값들을 다시 한번 확인해보시길 추천드립니다.
![](https://velog.velcdn.com/images/unani/post/92f9e8cf-347a-4348-a6ef-cf8d66adee8d/image.png)


3. S3 생성
S3는 AWS에서 제공하는 Simple Storage Service입니다. 
사이트의 컨텐츠가 올라갈 공간입니다.
리전을 다시 서울로 변경하고 AWS Console > Amazon S3 > 버킷 만들기를 차례대로 클릭합니다.
GCP와는 다르게 버킷명 규칙은 소문자,점(.),하이픈(-)으로 정해야 한다 등등의 규칙정도만 있습니다.
(GCP는 사용할 도메인명과 같아야 한다던가 그런 규칙들이 있습니다.)
ACL 비활성화, 모든 퍼블릭 엑세스 차단을 선택하고 버킷 만들기 버튼을 누릅니다.
정적웹서비스를 할 목적으로 S3 버킷을 만들고 있지만 S3에 퍼블릭 엑세스를 허용하면 
아무나 S3 데이터에 접근 할 수 있게 되기 때문에 예측하지 못한 비용이 발생할 수 있습니다.
엑세스 제어는 CloudFront에서 할 예정입니다.
![](https://velog.velcdn.com/images/unani/post/414ef545-d862-4baf-819e-4424c8504d04/image.png)
버킷을 생성하고 저는 HelloWorld가 출력되는 index.html 파일 하나를 올려두었습니다.
![](https://velog.velcdn.com/images/unani/post/111f3546-7273-47f0-b46c-9a7bc4140290/image.png)

3. CloudFront 배포 생성
S3는 일종의 cloud storage 역할만 하므로 엑세스 제어나 데이터 접근등은 가능하지만 단독으로 정적 웹사이트 서비스를 할 수는 없습니다.
특히 HTTPS 요청등을 처리 할 수는 없어서 CloudFront + S3로 구성하게 됩니다.
CloudFront는 CDN(Content Delivery Network) 서비스의 일종으로 각 지역에 있는 엣지 로케이션을 활용하여 컨텐츠를 캐싱하고 보다 빠른 서비스를 할 수 있도록 도와줍니다. CDN 및 CloudFront의 개념을 모르신다면 한번 찾아보시길 바랍니다.
AWS Console > CloudFront로 들어갑니다. CloudFront는 AWS의 글로벌 서비스이기에 자동으로 리전이 글로벌로 셋팅됩니다.
배포 생성 버튼을 누릅니다.
Origin domain 선택을 누르면 조금 전에 생성한 S3 버킷이 보입니다. 
또한 웹사이트로 사용하려면 버킷 엔드포인트 대신 S3 웹 사이트 엔드포인트를 사용하라는 안내문이 나오지만
S3에 퍼블릭 엑세스를 차단했으므로 웹 사이트 엔드포인트로 사용하게 되면 403 권한 에러가 발생하게 될 겁니다.
때문에 안내문은 무시합니다.
![](https://velog.velcdn.com/images/unani/post/0b1ffc37-0063-46f4-bad6-fd3ed8c6d2a2/image.png)
원본 엑세스 제어에서 원본 엑세스 제어 설정(권장)을 선택합니다.
![](https://velog.velcdn.com/images/unani/post/f643b164-e240-44ef-996b-42c0af0e137f/image.png)
origin access control 옆에 Create new OAC 버튼을 누릅니다.
OAC는 Origin access control의 약자로 기존 OAI방식의 단점을 극복하고 보안을 강화한 방식입니다.
![](https://velog.velcdn.com/images/unani/post/2b7eb2d2-fdf2-40e0-a6e8-c9c780502f3d/image.png)
Create 버튼을 누르면 기존 화면으로 돌아갑니다.
뷰어 영역에서 뷰어 프로토콜 정책을 Redirect HTTP to HTTPS를 선택합니다. HTTP로 요청이 들어오면 자동으로 HTTPS로 리다이렉트 시키겠다는 의미입니다.
![](https://velog.velcdn.com/images/unani/post/6fc6f43d-d96c-4de5-a833-657beb5c3da8/image.png)
Custom SSL certificate 부분의 셀렉트를 클릭하면 이미 버지니아북부 리전에 생성해놓은 인증서가 보입니다. 
선택합니다.
![](https://velog.velcdn.com/images/unani/post/2c9e08e7-d15c-41df-8fee-95ea6ccd4ff4/image.png)
배포 생성이 정상적으로 완료되면 정책복사 버튼이 나옵니다. 정책 복사 버튼을 누르면 정책이 클립보드에 저장됩니다.
또한 세부 정보 항목의 배포 도메인 이름을 적어둡니다.
이제 S3의 버킷 권한을 업데이트하러 이동합니다.
![](https://velog.velcdn.com/images/unani/post/93134900-e1a4-46f1-9821-8d92aebdd85a/image.png)
S3 버킷 페이지에서 권한으로 이동하여 버킷 정책 편집 버튼을 클릭하고 클립보드에 있는 정책을 붙여넣습니다.
혹시 편집 후 변경 사항 저장시 권한 충돌 오류가 발생한다면 S3 버킷의 퍼블릭 엑세스 차단을 잠깐 풀고 권한을 저장하고 다시 활성화 하시면 됩니다.
![](https://velog.velcdn.com/images/unani/post/2143517c-298b-4298-ad7f-8d1bc552916f/image.png)

4. 도메인의 CNAME 레코드 등록
이제 S3 및 CloudFront의 설정은 모두 끝났습니다.
AWS Console > Route 53 > DNS 관리로 이동해서 아까 등록한 도메인을 클릭합니다.
레코드 유형을 CNAME으로 설정하고 레코드 이름에 www, 값에는 조금 전에 CloudFront 배포 생성 후 적어놓은 배포 도메인 이름을 붙여넣습니다.
![](https://velog.velcdn.com/images/unani/post/90457c80-3b14-4ebb-ba3a-5086333fc310/image.png)
레코드가 정상적으로 등록되었습니다.
![](https://velog.velcdn.com/images/unani/post/e100f40a-b43a-4163-b2a4-d3f185f4e8ea/image.png)

5. 동작 확인
CloudFront는 글로벌 서비스임으로 배포까지 약간의 시간이 소요됩니다. (10분~20분?)
클라우드 프론트의 배포 도메인으로 접근하여 접근이 잘 되는지 확인합니다.
![](https://velog.velcdn.com/images/unani/post/427d67bc-9570-4601-836f-eac08721e2f1/image.png)
등록한 CNAME 레코드가 잘 반영이 되었는지 터미널에서 Dig 명령어로 확인합니다.
![](https://velog.velcdn.com/images/unani/post/ed018863-879e-48ba-afa7-8662d2e8096b/image.png)
도메인으로 접근하여 잘 동작하는지 확인합니다. Hello World!가 매우 반갑네요.
![](https://velog.velcdn.com/images/unani/post/d2eb23db-9e08-4bd1-a289-d5691e498ff2/image.png)
혹시 도메인으로 접근이 잘 안된다면 시간을 가지고 조금 기다려보세요 DNS의 반영시간은 예측하기 어렵습니다.
계속 반영이 안된다면 인증서 생성 항목부터 다시 한번씩 살펴보시길 바랍니다.