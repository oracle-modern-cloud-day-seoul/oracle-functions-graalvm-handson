# Oracle Functions with GraalVM on OCI Hands-On (Oracle Modern Cloud Day 2019의 Tech Hands-on Track)

![](images/header.png)

## Introduction
본 핸즈온 문서는 ....



본 핸즈온 문서는 Oracle Resource Manager와 Terraform Configuration을 사용하여 간단한 샘플 웹 애플리케이션 운영을 위한 배포 환경을 Oracle Cloud Infrastructure 환경에 자동으로 구성하고 배포하는 과정을 다루고 있습니다. 본 과정을 통해서 기본적인 오라클 클라우드 인프라 구성을 위한 서비스 리소스와 Terraform 코드를 통해 Oracle Cloud 인프라 구성 및 애플리케이션 배포를 자동화 해보는 경험을 해볼 수 있습니다.



## Objectives
* Oracle Cloud Infrastructure ....

## Required Artifacts
* 인터넷 접속 가능한 랩탑
* OCI (Oracle Cloud Infrastructure) 계정
* SSH Terminal (windows Putty, macOS Terminal 등)

## Client 접속 환경
ssh -i id_rsa opc@132.145.83.122
실습 환경 접속 정보 받기

## Serverless
### Cold start and Warm start

## Functions and GraalVM
### Oracle Functions Cloud Service


## Hands-On Steps

***

## **STEP 1**: GraalVM CE 설치
GraalVM CE를 설치하는 방법은 여러가지가 있습니다. GraalVM은 [GraalVM 다운로드 페이지](https://www.graalvm.org/downloads/)에서 패키징된 파일을 다운로드 받을 수 있습니다. GraalVM EE의 경우는 [Oracle에서 제공하는 페이지](https://www.oracle.com/downloads/graalvm-downloads.html)에서 다운로드 받을 수 있습니다.
macOS의 경우는 brew를 사용해서 설치할 수도 있습니다.

참고) macOS Homebrew를 통한 설치
```
$ brew cask install graalvm/tap/graalvm-ce
```

본 실습에서는 SDKMAN이라고 하는 SDK 관리툴을 사용하여 GraalVM을 설치합니다.

1. SDKMAN 설치
```
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

2. SDKMAN 에서 지원하는 Java 목록 확인
```
$ sdk list java
```

3. GraalVM 설치
```
$ sdk install java 19.2.1-grl
```

4. GraalVM 설치 확인
```
$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-20191008104205.buildslave.jdk8u-src-tar--b07)
OpenJDK 64-Bit GraalVM CE 19.2.1 (build 25.232-b07-jvmci-19.2-b03, mixed mode)
```

## **STEP 2**: OCIR (Oracle Container Infrastructure Registry) Login 정보
Oracle Function을 사용하기 위해서는 기본적으로 Docker를 활용하여 이미지를 생성하고, 이를 Docker Registry에 푸시 합니다. Docker Registry는 Oracle Cloud Infrastructure (이하 OCI)에서 제공하는 Oracle Container Infrastructure Registry (이하 OCIR)를 사용하게 됩니다. OCIR 접속을 위한 정보는 Registry URL, Username, Password로 각각의 정보를 얻는 과정은 다음과 같습니다.

1. Registry URL
- 기본 주소 포멧은 **{region_code}.ocir.io** 형태이며, regison_code는 https://docs.cloud.oracle.com/iaas/Content/General/Concepts/regions.htm 에서 확인가능합니다.
- 본 실습에서는 애시번 (iad)리전의 Registry를 사용합니다.
    - iad.ocir.io  
    <font color='red'>(Registry URL은 Function 설정시에도 필요하므로 메모합니다!)</font>

2. Username
- 기본 OCIR 사용자 아이디 포멧은 **{tenancy_namespace}/{oci계정}** 입니다.
- tenancy_namespace는 OCI Console 로그인 후 우측 상단의 사용자 아이콘 클릭 > Tenancy 클릭하면 확인할 수 있는 **Object Storage Namespace** 값 입니다.
  ![](images/oci_tenancy_namespace.png)
- oci계정은 OCI Console 로그인 후 우측 상단의 사용자 아이콘 클릭 > Profile 바로 밑의 사용자 아이디를 클릭하면 확인할 수 있습니다.
  ![](images/oci_username.png)
- OCIR Username 예시: idsufmye3lml/oracleidentitycloudservice/donghu.kim@oracle.com  
<font color='red'>(Tenancy Namespace는 Function 설정시에도 필요하므로 메모합니다!)</font>

3. Password
- OCIR 로그인을 위한 패스워드는 OCI Console에서 토큰을 임시 발행하여 이를 활용합니다.
- 토큰 발행은 OCI Console 로그인 후 우측 상단의 사용자 아이콘 클릭 > Profile 바로 밑의 사용자 아이디를 클릭 > 좌측 Auth Tokens 클릭 > Generate Token 클릭 > Description에 **ocir-token** 입력 후 Generate Token 클릭하여 생성된 토큰을 복사합니다.  
<font color='red'>(토큰은 Function 설정시에도 필요하므로 메모합니다!)</font>

## **STEP 2**: Docker Login
docker login 명령어를 사용하여 OCIR에 로그인합니다.
```
$ docker login {Registry URL} --username {Username} --password '{Password}'
```
접속 예시
```
$ docker login icn.ocir.io --username idsufmye3lml/oracleidentitycloudservice/donghu.kim@oracle.com --password 'hT{+t3KnuF.5x42a(>l)'

WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/admin/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

## **STEP 3**: Fn Project CLI 설정
실습에서는 미리 설치되어 있는 Fn Project CLI를 사용합니다. Fn Project CLI 설치 후 Fn Project CLI에서 필요로 하는 정보를 Context로 생성해서 구성해야 합니다.
> Fn Project CLI 설치에 대한 가이드는 아래 페이지를 참고하세요.  
> Fn Project CLI Install Guide (https://fnproject.io/tutorials/install/)

1. 우선 Fn Project CLI 설치 확인을 합니다.
    ```
    $ fn version
    ```

2. Context를 생성합니다.
    기본 사용법은 다음과 같습니다.
    ```
    $ fn create context <my-context> --provider oracle
    ```

    <my-context>는 관리할 Context의 이름으로 실습에서는 **helloworld**로 지정하여 생성합니다. 
    ```
    $ fn create context helloworld --provider oracle

    Successfully created context: helloworld
    ```

3. Fn Project CLI에서 위에서 생성한 Context를 사용하도록 설정합니다.
    ```
    $ fn use context helloworld
    ```

4. 생성한 Context를 업데이트 합니다. 필요한 정보는 OCI CLI의 Profile, function 서버의 API URL, Compartment OCID, Container Registry로 먼저 OCI CLI의 Profile을 업데이트 합니다.
    Fn Project Context에 OCI Profile 업데이트는 다음과 같이 실행합니다.
    ```
    $ fn update context oracle.profile <profile-name>
    ```

    oci-cli의 profile은 다음과 같이 확인할 수 있습니다. **[DEFAULT]** 부분이 profile명입니다.
    ```
    $ cat ~/.oci/config

    [DEFAULT]
    user=ocid1.user.oc1..aaaaaaaalpieyqquaaneneuyiifrtfbzwcr3hqd7tqfoobwq7xr4jv5pfz3a
    fingerprint=48:1a:98:8c:cd:f6:63:4b:fb:4d:8d:26:44:aa:37:f6
    key_file=/Users/DonghuKim/.oci/oci_api_key.pem
    tenancy=ocid1.tenancy.oc1..aaaaaaaa6ma7kq3bsif76uzqidv22cajs3fpesgpqmmsgxihlbcemkklrsqa
    region=ap-seoul-1
    ```

    실제 profile[DEFAULT]을 적용하여 업데이트합니다.
    ```
    $ fn update context oracle.profile DEFAULT
    ```

5. OCI에서 제공하는 Function API URL을 업데이트 합니다.
    ```
    $ fn update context api-url https://functions.us-ashburn-1.oraclecloud.com
    ```

    > Function API Url은 각 Region별로 URL이 다르게 되어 있습니다. 배포하고자 하는 리전에 맞춰서 Function API URL을 사용합니다.

6. Compartment OCID를 업데이트 합니다. OCI Console에서 생성한 Compartment OCID는 메뉴 > Identity > Compartments 선택 후 생성한 Compartment 클릭하면 확인할 수 있습니다.
  ![](images/oci_my_compartment_ocid.png)

    기본 사용법은 다음과 같습니다.
    ```
    $ fn update context oracle.compartment-id <compartment-ocid>
    ```

    실제 Compartment OCID를 적용한 예시입니다.
    ```
    $ fn update context oracle.compartment-id ocid1.compartment.oc1..aaaaaaaanojzru4tvrayjwezor2dlbo2um25xodb5bz2zp4kyx3nj7xgax6a
    ```

7. Oracle Container Registry URL과 Repository 설정입니다.
    기본 사용법은 다음과 같습니다.
    ```
    $ fn update context registry <region-code>.ocir.io/<tenancy-namespace>/<repo-name>
    ```

    실제 region-code, tenancy-namespace, repo-name을 적용한 예시입니다. repo-name은 helloworld로 통일합니다.
    ```
    $ fn update context registry iad.ocir.io/idsufmye3lml/helloworld
    ```

8. 설정된 context의 내용을 확인합니다.
    ```
    $ cat $HOME/.fn/contexts/helloworld.yaml

    api-url: https://functions.us-ashburn-1.oraclecloud.com
    oracle.compartment-id: ocid1.compartment.oc1..aaaaaaaanojzru4tvrayjwezor2dlbo2um25xodb5bz2zp4kyx3nj7xgax6a
    oracle.profile: DEFAULT
    provider: oracle
    registry: iad.ocir.io/idsufmye3lml/helloworld
    ```

## **STEP 5**: VCN 생성

## **STEP 6**: Function Application 생성

## **STEP 7**: 일반 Java Function 생성 및 배포
```
$ fn init --runtime java helloworld-func
```

## **STEP 4**: Hello World Application 생성



## **STEP 6**: Native-Image Java FUnction 생성 (GraalVM)

## **STEP 7**: Function 호출 테스트






### Compartment 만들기
<font color='red'>이미 만들어진 Compartment가 있을 경우 이 단계는 건너뜁니다.</font>
> ***Compartment***   
> 모든 OCI 리소스는 특정 Compartment에 속하게 되며 Compartment 단위로 사용자들의 접근 정책을 관리할 수 있습니다. 처음에는 Root Compartment가 만들어지며, Root Compartment 하위에 추가 Compartment를 생성할 수 있습니다. OCI 클라우드 리소스를 쉽게 관리하기 위한 일종의 폴더 개념이라고 생각하면 됩니다. 부서나 프로젝트등을 고려해서 Compartment를 구성하여 해당 Compartment별로 세부적인 권한을 부여할 수 있습니다.

먼저 OCI Console에 로그인합니다.

1. https://console.us-ashburn-1.oraclecloud.com 접속 후 Tenant 입력 > 

## 참고
https://medium.com/criciumadev/serverless-native-java-functions-using-graalvm-and-fn-project-c9b10a4a4859
https://medium.com/thundra/mastering-java-cold-start-on-aws-lambda-volume-1-21c30ce378b7
https://royvanrijn.com/blog/2018/09/part-2-native-microservice-in-graalvm/ : SubstrateVM