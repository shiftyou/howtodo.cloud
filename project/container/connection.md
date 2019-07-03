# 환경 및 프로그램

도커 및 쿠베너티스의 테스트를 위하여 필요한 프로그램과 환경이 있습니다. 

- docker : 컨테이너 환경 및 프로그램
- git : 소스 관리 프로그램
- kubectl : 쿠베르네테스 관리 프로그램

실습을 위하여 미리 VM 을 만들어 놓았습니다.


# VM에 접속하기

putty 를 이용하여 VM에 접속합니다. 프로그램은 [여기]( https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)에서 받을 수 있습니다.

putty를 시작하면 VM에 접속하기 위한 환경을 설정해야 합니다.

1. 먼저 좌측의 Connection > SSH > Auth 를 선택합니다. 그리고 Private key file for authenticate 에서 전달받은 privatekey.ppk 를 선택합니다.

    ![setting](https://jonggyoukim.github.io/container/images/putty_setting.png)

1. 다시 좌측에서 최상위의 Session 을 선택하고 다음을 입력합니다.
    - Host Name : 129.213.116.199
    - Saved Sessions : cloud

    그리고 Save 를 선택하여 저장합니다.

    ![setting2](https://jonggyoukim.github.io/container/images/putty_setting2.png)

1. 이제 하위의 Open 을 누릅니다. 그러면 다음과 같이 접속이 됩니다.
    
    ![putty](https://jonggyoukim.github.io/container/images/putty1.png)

1. 아이디는 user1 ~ user9 가 있습니다. 자신이 받은 아이디를 사용합니다. 아이디를 입력하면 자동으로 로그인이 됩니다.
    ![putty](https://jonggyoukim.github.io/container/images/putty2.PNG)

1. 접속 완료하셨습니다.
