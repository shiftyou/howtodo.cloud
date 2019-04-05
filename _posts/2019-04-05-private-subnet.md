---
layout: post
title:  "private subnet 에서 JCS/SOACS 프로비저닝"
categories: [oci]
tags: [private subnet]
---
# private subnet 에서 JCS/SOACS 프로비저닝
아래는 private subnet에 OCI Database를 이용하는 JCS 인스턴스를 만드는 과정입니다.
이와 같은 방법으로 SOACS 인스턴스를 private subnet 에서 만드는 방법도 소개합니다.

## JCS 프로비저닝 JSON 파일 얻기 

JCS 프로비저닝 단계의 마지막에서 json 파일을 얻을 수 있습니다.
다음의 절차로 json 파일을 얻습니다.

1. Oracle Java Cloud Service 인스턴스 생성 버튼을 눌러 Java 인스턴스를 생성합니다.

1. 프로비저닝을 하기 위해서 정보를 넣습니다.
    
    - instance name : JCS 인스턴스의 이름
    - Region : 프로비저닝 할 리젼 선택
    - Availability Domain : 도메인 선택
    - Subnet : `public subnet 을 선택 (private subnet은 보이지 않음)`
    
    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/jcs2.png)

1. 항목을 채웁니다.

    - Databaes Type : Oracle Cloud Infrastructure Database
    - Compartment Name : 선택합니다.
    - Database Instance Name : 선택합니다.
    - Administrator User Name : SYS
    - Passworld : 패스워드

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/jcs3.png)
    
1. json 파일을 다운로드 받습니다.

    화면 우측의 ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/jcs5.png)를 눌러 JCS 파일을 다운로드 받습니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/jcs4.png)

    화면 왼측의 "취소"를 눌러 프로비저닝 단계를 취소합니다.

## JSON 파일 수정하기

다운로드 받은 JSON 파일은 다음과 같습니다.

~~~json
{
    "subnet": "ocid1.subnet.oc1.iad.aaaaaaaafsqkriuuisvrrap62wlyjuns3xkfe5twdxwltu5nnbowmue37eoa",
    "enableAdminConsole": "false",
    "components": {
        "WLS": {
            "domainPartitionCount": "0",
            "adminUserName": "weblogic",
            "isOciRacDb": "false",
            "dbType": "OCINativeDB",
            "dbaPassword": "<Fill_Here>",
            "dbaName": "SYS",
            "connectString": "//jgdb-scan.dmzpivatedataba.np2020.oraclevcn.com:1521/jgdb_iad1gw.dmzpivatedataba.np2020.oraclevcn.com",
            "sampleAppDeploymentRequested": "true",
            "clusters": [
                {
                    "serverCount": "1",
                    "shape": "VM.Standard1.1",
                    "clusterName": "myjcs_cluster",
                    "type": "APPLICATION_CLUSTER"
                }
            ],
            "adminPassword": "<Fill_Here>"
        }
    },
    "availabilityDomain": "PdJy:US-ASHBURN-AD-1",
    "useOAuthForStorage": "false",
    "edition": "EE",
    "vmPublicKeyText": "ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAiwuQVZUskpVDu72CqbinJhwxM3tZ8+lJ1/YPNAsduzCJqzrQZNDPLEWtZOfZjYGPvUr7lP+ruF8D4vO14hjOsHipchkkn765NmX94HX2m0uy9yocs/vaXBxu/3+jBR/wplusUJr8qr+r5LZctvpOhLYjAJE13vzR+RoYYBVNxU2rVulI4LJ7eePFudfcnTQ18TrTjZTo7Jpc//aH21xYMOtcAMS5aqmNN5RTWubzNti8hr37paKGCQM8ARFtv0yB7y5sBBtBetBG5VsKHEpk3ztreJkhfgS/uTGT7Jqv8PKMB0Kfd02yZpNons9LZp4U9yiWng3n9knO4qSwxqY48w== rsa-key-20190222",
    "serviceLevel": "PAAS",
    "serviceName": "myjcs",
    "notificationEmail": "jonggyou.kim@oracle.com",
    "backupDestination": "NONE",
    "serviceVersion": "12cRelease213",
    "enableNotification": "true",
    "meteringFrequency": "HOURLY",
    "serviceDescription": "jcs test",
    "region": "us-ashburn-1",
    "subscriptionId": "7134652",
    "loadBalancerOption": "NONE",
    "isBYOL": "true"
}
~~~
여기에서 수정할 부분은 `subnet` 과 `<Fill_Here>`입니다.
- subnet : private subnet 을 넣어줍니다.
- <Fill_Here> : Database 의 패스워드 및 JCS의 패스워드를 넣어줍니다.



## JSON 파일로 프로비저닝 하기

json 파일을 이용하여 jcs 프로비져닝을 합니다. 이를 위해 필요한 것이 psm cli 입니다.

다운로드 및 설치는 다음을 참조하시면 됩니다.

1. [Downloading the CLI from the Oracle Cloud User Interface](https://docs.oracle.com/en/cloud/paas/java-cloud/pscli/downloading-cli-your-service-user-interface.html)
1. [Installing the Command Line Interface](https://docs.oracle.com/en/cloud/paas/java-cloud/pscli/installing-command-line-interface.html)
1. [Configuring the Command Line Interface](https://docs.oracle.com/en/cloud/paas/java-cloud/pscli/configuring-command-line-interface.html)

psm을 활용하여 다음을 수행하여 프로비져닝 합니다.
~~~
$ psm jcs create-service -c service_payload_myjcs.json

Message : Submitted job to create service [myjcs] in domain [dics-123l1j4...123123]
Job ID  : 688991105
~~~
프로비져닝이 성공하면 cloud console 에서 해당 인스턴스가 보입니다.

# JCS Console 접근하기

private subnet 으로 만들어진 jcs는 접근이 불가합니다. 그래서 LoadBalancers를 이용하여 접근하도록 해야 합니다.

1. OCI의 Networking > Load Balancers 를 클릭합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/lb1.png)

1. Load Balancers를 Public 으로 만듭니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/lb2.png)

1. Backend Sets을 만들고 80 포트를 해당 JCS의 IP에 연결합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/lb3.png)

1. Listeners에 80 포트를 만들어줍니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/lb4.png)

1. Load Balancers의 IP를 이용하여 sampe-app에  접근합니다.

    ![](https://raw.githubusercontent.com/jonggyoukim/oci/master/images/lb5.png)

# SOACS를 private subnet 에서 프로비져닝 하기
JCS의 json 파일을 얻는 방법과 마찬가지로 json 파일을 얻습니다.

~~~json
{
    "enableAdapters": "false",
	"subnet": "ocid1.subnet.oc1.iad.aaaaaaaamo3awfr5jb3o6tef7pbr6e4qedp3ti6mj7g7ecnoh6v4wel3nz2q",
	"enableAdminConsole": "true",
	"components": {
		"WLS": {
			"adminUserName": "weblogic",
			"shape": "VM.Standard2.1",
			"isOciRacDb": "false",
			"dbType": "OCINativeDB",
			"dbaPassword": "<Fill_Here>",
			"dbaName": "SYS",
			"connectString": "//jgdb-scan.dmzpivatedataba.np2020.oraclevcn.com:1521/jgdb_iad1gw.dmzpivatedataba.np2020.oraclevcn.com",
			"managedServerCount": "1",
			"adminPassword": "<Fill_Here>"
		}
	},
    "availabilityDomain": "PdJy:US-ASHBURN-AD-1",
    "purchasePack": "soaosbb2b",
	"edition": "SUITE",
	"vmPublicKeyText": "ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAiwuQVZUskpVDu72CqbinJhwxM3tZ8+lJ1/YPNAsduzCJqzrQZNDPLEWtZOfZjYGPvUr7lP+ruF8D4vO14hjOsHipchkkn765NmX94HX2m0uy9yocs/vaXBxu/3+jBR/wplusUJr8qr+r5LZctvpOhLYjAJE13vzR+RoYYBVNxU2rVulI4LJ7eePFudfcnTQ18TrTjZTo7Jpc//aH21xYMOtcAMS5aqmNN5RTWubzNti8hr37paKGCQM8ARFtv0yB7y5sBBtBetBG5VsKHEpk3ztreJkhfgS/uTGT7Jqv8PKMB0Kfd02yZpNons9LZp4U9yiWng3n9knO4qSwxqY48w== rsa-key-20190222",
	"serviceLevel": "PAAS",
	"serviceName": "jgsoa",
	"notificationEmail": "jonggyou.kim@oracle.com",
	"serviceVersion": "12cRelease213",
	"cloudStorageUser": "jonggyou.kim",
	"cloudStorageContainer": "https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/hmm21/paasbackup",
	"enableNotification": "true",
	"meteringFrequency": "HOURLY",
	"cloudStoragePassword": "<Fill_Here>",
	"region": "us-ashburn-1",
	"subscriptionId": "7134652",
	"provisionOTD": "false",
	"isBYOL": "true"
}
~~~

subnet 과 <Fill_Here>를 채워줍니다.

JCS와 마찬가지로 psm을 이용하여 프로비저닝 합니다.
~~~
$ psm soa create-service -c service_payload_mysoa.json
~~~


    
