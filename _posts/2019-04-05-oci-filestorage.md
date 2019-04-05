---
layout: post
title:  "OCI의 File Storage를 Compute Instance에 연결하는 방법"
categories: [oci]
tags: [oci, storage]
---
# OCI의 File Storage를 Instance에 연결하는 방법

1. OCI의 File Storage 를 클릭합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs1.png)

1. Create File System 을 클릭합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs2.png)

1. File System Information 의 Edit Details 를 눌러 이름을 바꿉니다. (예:logfile)

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs3.png)
    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs4.png)

1. 아래의 Mount Target Information 눌러 Subnet 을 선택합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs6.png)

1. Create 를 눌러 File Storage 를 생성합니다.<br>생성이 완료되면 화면 아래 오른쪽의 Mount Commands 를 누릅니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs5.png)

1. 실행할 명령을 확인합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs7.png)

1. Subnet의 Security List를 다음의 정보를 추가 합니다.

    - TCP: 2048~2050, 111 포트
    - UDP: 2048, 111 포트

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/fs8.png)

1. File Storage 를 붙일 instance에 ssh 로 접속을 합니다.

1. 순서대로 명령을 수행합니다.

    필요 라이브러리를 설치합니다.
    ~~~sh
    [opc@ohs-instance ~]$ sudo yum install nfs-utils
    Loaded plugins: langpacks, ulninfo
    ol7_UEKR5                                                | 1.2 kB     00:00
    ol7_addons                                               | 1.2 kB     00:00
    ol7_developer                                            | 1.2 kB     00:00
    ol7_developer_EPEL                                       | 1.2 kB     00:00
    ol7_latest                                               | 1.4 kB     00:00
    ol7_optional_latest                                      | 1.2 kB     00:00
    ol7_software_collections                                 | 1.2 kB     00:00
    (1/4): ol7_latest/x86_64/updateinfo                        | 852 kB   00:00
    (2/4): ol7_optional_latest/x86_64/updateinfo               | 664 kB   00:00
    (3/4): ol7_optional_latest/x86_64/primary                  | 2.8 MB   00:00
    (4/4): ol7_latest/x86_64/primary                           |  12 MB   00:00
    ol7_latest                                                          12465/12465
    ol7_optional_latest                                                   9598/9598
    Package 1:nfs-utils-1.3.0-0.61.0.1.el7.x86_64 already installed and latest version
    Nothing to do
    [opc@ohs-instance ~]$
    ~~~

    마운트 할 디렉토리를 만듭니다.
    ~~~sh
    [opc@ohs-instance ~]$ sudo mkdir -p /mnt/logfile
    [opc@ohs-instance ~]$
    ~~~

    마운트 합니다.
    ~~~sh
    [opc@ohs-instance ~]$ sudo mount 10.150.100.7:/logfile /mnt/logfile
    [opc@ohs-instance ~]$
    ~~~

    마운트 된 상태를 확인합니다.
    ~~~sh
    [opc@ohs-instance ~]$ df -h
    Filesystem             Size  Used Avail Use% Mounted on
    devtmpfs               7.2G     0  7.2G   0% /dev
    tmpfs                  7.3G     0  7.3G   0% /dev/shm
    tmpfs                  7.3G   17M  7.2G   1% /run
    tmpfs                  7.3G     0  7.3G   0% /sys/fs/cgroup
    /dev/sda3               39G  7.4G   31G  20% /
    /dev/sda1              200M  9.7M  191M   5% /boot/efi
    tmpfs                  1.5G     0  1.5G   0% /run/user/54321
    tmpfs                  1.5G     0  1.5G   0% /run/user/1000
    10.150.100.7:/logfile  8.0E     0  8.0E   0% /mnt/logfile
    [opc@ohs-instance ~]$
    ~~~
    
    완료하셨습니다.
