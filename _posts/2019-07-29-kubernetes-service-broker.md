
# << 작성중 >>

Service Broker

OSB(공개 서비스 브로커) API를 통해서 구현되는 서비스 브로커는 OCI위에 리소르를 만들고 해당 권한을 관리하고 OCI서비스르 제공해주는 서비스이다. 쿠버네티스 클러스터의 애플리케이션들은 서비스 브로커를 통해서 필요한 OCI의 서비스를 사용할 수 있다.서비스 브로커를 통해서 OCI의 리소스를 만들고 Kubernetes 클러스터 내에서 OCI의 서비스를 사용할 수 있게 해 준다.

서비스 브로커는 Kubernetes 클러스터에서 Service Category로 설치가 된다. Service Catalog 에 OCI를 위한 서비스 브로커가 설치가 되고 Kubernetes 클러스터 상에서 해당 서비스를 사용할 수가 있다.

![](https://cdn.app.compendium.com/uploads/user/e7c690e8-6ff9-102a-ac6d-e4aebca50425/41d1c169-5ecc-4442-ab54-fc8d9cb3cdc6/Image/9a2cd983e25311180b4bf604fd7d58d9/svc_brkr_arch_4.jpg)

현재 OCI 서비스 브로커는 다음의 서비스들을 사용한다.
- Autonomous Transaction Processing
- Autonomous Data Warehouse
- Object Storage
- Streaming

쿠버네티스 클러스터에 서비스 브로커를 추가한 다음, Open Service Broker APIs를 통하여 kubectl