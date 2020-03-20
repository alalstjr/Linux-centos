# 윈도우즈 10 Hyper-V 사용중지

VMware Player and Device/Credential Guard are not compatible. VMware Player can be run after disabling Device/Credential Guard. Please visit http://www.vmware.com/go/turnoff_CG_DG for more details.

* 발생 원인

Windows 10부터는 OS 내에서 자체 가상화를 지원하고 있는데 VMware 와 충돌으로 인해 문제가 발생되고 있는 것으로 생각됩니다.

1. 실행 창에서 appwiz.cpl 을 실행합니다.
2. "프로그램 및 기능"에서 "Windows 기능 켜기/끄기"를 클릭합니다.
3. "Windows 기능"에서 Hyper-V 관련 폴더가 체크되어 있는데 이 부분을 체크 해제합니다.

# VM 머신 설치

1. Create a New Virtual Machine

리눅스 배포판 iso 파일을 로드하는 부분입니다.
Installer disc image file (iso) 클릭하여 CentOS ISO 경로를 설정해 줍니다.

Store virutal disk as a single file : 하드디스크를 파일 하나에 모두 저장
Split virutal disk into multiple files : 파일을 여러 개로 분할해서 저장

각자 장단점이 있는데 전자는 파일을 1개로 관리하면 이동할 때 쉽습니다.

Store virtual disk as a single file 클릭 후 `Maximum disk size (GB): 은 40 으로 설정`합니다.

Edit virtual machine settings 를 눌러서 마저 작업해주도록 합시다.

가장 먼저 보이는 `Memory 부분을 넉넉하게 2GB(2048MB)` 로 잡아줍니다.

# VM 머신 실행 

- 키보드 설정 클릭후 `영어(미국)` 을 추가합니다.
- 소프트웨어 선택 `클릭 후 개발 및 창조를 위한 워크스테이션`으로 설정합니다.

- 설치 목적지 클릭합니다.
    - 윈도우즈에서도 파티션을 분할하듯이 리눅스에서도 파티션을 나눠주어야 합니다.
    - 저장소 구성에서 `Custom 체크`합니다.
    - `여기를 클릭하여 자동으로 생성합니다.` 를 클릭합니다.
    - 리눅스에서는 윈도우즈와 다르게 C 드라이브, D 드라이브 같은 개념이 없습니다.
    - Oracle DB 같은 경우 swap 공간이 부족하면 안되므로 / 와 swap 을 싹 다 지우고 새로 만들겠습니다.
        - /boot : 부팅에 관련된 정보를 저장하는 디렉토리
        - / : 루트 디렉토리
        - swap : 메모리가 부족할 때 추가 작업을 위한 공간(윈도우즈로 말하면 가상 메모리)
    - 플러스 버튼으로 `SWAP 용량 5GB 로 생성`합니다.
    - 루트 디렉토리(/) 를 마운트 지점으로 하고 용량 부분은 비워둡시다
-  네트워크 & 호스트 이름
    - `끔 -> 켬` 으로 설정합니다.
- 설치되는 동안  ROOT 암호 설정
- 사용자 생성

# JAVA 설치

- 터미널창을 열어줍니다.
- yum으로 설치 가능한 jdk 목록 확인
    - $ `yum list java*jdk-devel` 
    - $ `yum install {java-jdk-name}`
    - `which java` JAVA 설치 경로 확인
    - `rpm -qa | grep openjdk` 설치된 JAVA 버전 확인

# Tomcat 설치

 - 톰캣 (Apache Tomcat) 7 을 관리할 계정과 그룹을 생성합니다.
    - groupadd tomcat
    - useradd -g tomcat -m -d /home/tomcat -s /bin/bash tomcat
    - id tomcat
- http://tomcat.apache.org/
    - download -> tar.gz
- download file 을 '/' 으로 가져와서 압축 풀기
    - tar -xzf {tar.gz}
    - mv apache-tomcat /tomcat8 새로운 폴더를 만들어 이동하기

- useradd 옵션
    - -g 사용자 계정의 GID 지정 
    - -M 사용자의 홈 디렉토리를 생성하지 않음 
    - -d 사용자의 홈 디렉토리를 지정
    - -s 로그인 시 사용할 기본 쉘 지정 
    - -u 사용자 계정의 UID 지정 
    - -g 사용자 계정의 GID 지정 
    - -e 사용자의 계정 만기일 지정 
    - -f 사용자의 계정 유효일 지정

- 압축 풀기 옵션
    - -c 파일을 tar로 묶음
    - -p 파일 권한을 저장
    - -v 묶거나 파일을 풀 때 과정을 화면으로 출력
    - -f 파일 이름을 지정
    - -C 경로를 지정
    - -x tar 압축을 풂
    - -z gzip으로 압축하거나 해제함

- 잠시동안 외부의 접속을 허용하도록 방화벽을 끄도록 합니다 `systemctl stop firewalld`
- 영구 해제(리부팅 이후에도 작동하지 않도록 설정) `systemctl disable firewalld`
- setenforce permissive
    - vi /etc/sysconfig/selinux
        - i, a 클릭 후 수정모드
        - 수정 완료 후 esc 클릭 후 : 클릭 z 입력하여 저장합니다.

- cd tomcat8 위치로 이동합니다.
    - ls bin 디렉토리 확인하면 리눅스는 실행파일을 startup.sh 로 실행합니다.
- 어느 폴더에 있던 시작 종료를 할 수 있도록 `명령어를 생성`합니다.
    - [root@server ~] 접속하는 계정의 홈 경로로 이동합니다.
    - vi $HOME/ .bash_profile
- cd tomcat8 위치로 이동합니다.
    - `/tomcat8/bin/startup.sh` 실행합니다.
- ps -ef | grep /tomcat8/bin 명령어로 프로세스가 올라가있는것을 확인합니다.
- web 자료는 webapp 폴더에 위치합니다.
- cd webapp/ROOT/index.jsp 를 읽어서 실행하는 것입니다.

## 참고

https://jeongyd.tistory.com/22
https://www.youtube.com/watch?v=FUNyYlhtlHA