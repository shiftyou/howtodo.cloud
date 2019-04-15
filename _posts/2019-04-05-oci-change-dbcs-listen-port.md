---
layout: post
title:  "DBCS 리슨포트 변경 및 JCS 프로비저닝"
categories: oci
tags: dbcs jcs
---
# DBCS 리슨포트 변경 및 JCS 프로비저닝

# DBCS 프로비저닝

일반적으로 DBCS를 프로비저닝 하는 방법과 동일하다.
몇가지 제약사항을 두어서 프로비저닝 한다.

1. private subnet 으로 지정한다.
2. DBCS를 만들때 Public Key를 넣도록 한다.


# DBCS 서버에 접근하기

DBCS를 private subnet 으로 만들었기 때문에 인터넷에서 바로 접근이 불가능하다. 그래서 Compute Instance를 public subnet으로 만들고 DBCS 인스턴스에 접근이 가능하다.

1. Compute Instance에 접근하기고 .ssh 디렉토리 만들기

    ssh 로 public ip를 가지는 compute instance에 접속하고 .ssh 디렉토리를 만든다.
    ~~~
    mkdir ~/.ssh
    ~~~

1. private key를 id_rsa 이름으로 복사한다.
    
    scp 를 이용하여 복사를 하면 된다.
    ~~~
    scp -i <private_key_file_for_compute> <private_key_file_for_dbcs> opc@<public_compute_instance_ip>:/home/opc/.ssh/id_rsa
    ~~~

1. ssh를 이용하여 compute instance 에서 DBCS 서버에 접근한다.
    
    ~~~
    ssh opc@<dbcs_ip>
    ~~~
    접속이 되는지 확인한다.

# DBCS 리스너 포트 변경하기 

DBCS의 리스너 포트를 변경하기 위하 다음과 같은 절차를 수행한다.
1. 변경을 하기 위하여 grid 계정으로 스위치 한다.
    
    ~~~
    sudo su - grid
    ~~~
   
1. 리스너를 내란다.

    ~~~
    lsnrctl stop LISTENER
    ~~~
    
1. 리스너가 내려갔는지 확인한다.

    ~~~
    lsnrctl status
    ~~~

1. 리스너 포트를 변경한다. (기존 1521 에서 1527로 변경)

    ~~~
    srvctl modify listener -l LISTENER -p 1527
    ~~~

1. 다시 리스너를 시작한다.

    ~~~
    lsnrctl start LISTENER
    ~~~

1. 리슨하고 있는지 시스템으로 살펴본다.

    ~~~
    netstat -na |grep 1527
    ~~~

1. opc 계정으로 돌아간다.

    ~~~
    exit
    ~~~

1. 오라클을 재시작 한다.

    ~~~
    sudo su - oracle
    sqlplus "/as sysdba"
    shutdown immediate
    startup
    ~~~

    
# 방화벽 열기

DBCS의 리눅스에는 방화벽이 설치되어 있다. Linux 7 이전이라 iptable을 사용한다.

1521 포트는 DBCS의 기본 포트라 시스템에서 방화벽에서 설정해 두었으며 새로 설정한 1527 포트를 설정해 주어야 한다.

1. /etc/sysconfig/iptables 파일을 수정한다.
    
    ~~~
    vi /etc/sysconfig/iptables
    ~~~

2. 1521 이 포함된 라인을 복사해서 1527로 변경한다.

    ~~~
    -A INPUT -p tcp -m state --state NEW -m tcp --dport 1521 -j ACCEPT -m comment --comment "Required for access to Database Listener, Do not remove or modify. "

    -A INPUT -p tcp -m state --state NEW -m tcp --dport 1527 -j ACCEPT -m comment --comment "Required for access to Database Listener, Do not remove or modify. "
    ~~~

3. 방화벽을 재기동 한다.

    ~~~
    sudo service iptables restart
    ~~~


# 접속 테스트 하기

1. 만들어진 DBCS를 선택하여 `DB접속`를 클릭한다.
    
    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/dbcs1.png)

1. `쉬운접속`의 `접속문자열`을 복사한다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/dbcs2.png)

    복사된 문자열의 예는 다음과 같다.
    ~~~
    db.privateregion.jgcluster1.oraclevcn.com:1521/orcl_iad19d.privateregion.jgcluster1.oraclevcn.com
    ~~~

1. sqlplus를 이용하여 테스트 한다.

    접속문자열 중 포트를 `'1521'` 에서 `'1527'`로 변경한다.
    
    ~~~sh
    splqlus system/<패스워드>@db.privateregion.jgcluster1.oraclevcn.com:1527/orcl_iad19d.privateregion.jgcluster1.oraclevcn.com
    ~~~

1. 접속 확인한다.

# JCS 프로비저닝 하기

기존의 private subnet 에서 JCS 프로비저닝 하는 방법과 동일하다.

다만, 위 테스트와 같이 DBCS 커넥션 하는 문자에서 1521을 1527로 수정하면 된다.

~~~json
"connectString": "//db-scan.privateregion.jgcluster1.oraclevcn.com:1527/pdb1.privateregion.jgcluster1.oraclevcn.com",
~~~

~~~json
{
    "subnet": "ocid1.subnet.oc1.iad.aaaaaaaa4odkf2zic6sfokhaxshsdff7bkiaxcmbgrru6b72yzi5zpqfhedq",
    "enableAdminConsole": "false",
    "components": {
        "WLS": {
            "domainPartitionCount": "0",
            "adminUserName": "weblogic",
            "isOciRacDb": "false",
            "dbType": "OCINativeDB",
            "dbaPassword": "<DB 패스워드>",
            "dbaName": "SYS",
            "pdbServiceName": "pdb1",
            "connectString": "//db-scan.privateregion.jgcluster1.oraclevcn.com:1527/pdb1.privateregion.jgcluster1.oraclevcn.com",
            "sampleAppDeploymentRequested": "true",
            "clusters": [
                {
                    "serverCount": "1",
                    "shape": "VM.Standard2.1",
                    "clusterName": "jg-db1527_cluster",
                    "type": "APPLICATION_CLUSTER"
                }
            ],
            "adminPassword": "<JCS 패스워드>"
        }
    },
    "availabilityDomain": "fttO:US-ASHBURN-AD-2",
    "edition": "EE",
    "vmPublicKeyText": "ssh-rsa AAAAB3NzaC1yc2EAAA...rsa-key-20190222",
    "serviceLevel": "PAAS",
    "serviceName": "jg-db1527",
    "notificationEmail": "jonggyou.kim@oracle.com",
    "tags": [
        {
            "value": "jonggyou.kim@oracle.com",
            "key": "MC_TEAM"
        }
    ],
    "backupDestination": "NONE",
    "serviceVersion": "12cRelease213",
    "enableNotification": "true",
    "meteringFrequency": "HOURLY",
    "serviceDescription": "hmm test by jonggyou.kim@oracle.com",
    "region": "us-ashburn-1",
    "subscriptionId": "7216493",
    "loadBalancerOption": "NONE",
    "isBYOL": "true"
}
~~~

PSM으로 명령을 수행하면 리슨포트가 기본 1521이 아닌 private dbcs를 사용하는 private jcs를 프로비저닝 할 수 있다.
