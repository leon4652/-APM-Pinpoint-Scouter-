# Scouter Setup Manual

## 구축 환경 Diagram과 Flow

### 전체 다이어그램
 ![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/01e8d94b-8b6e-459b-9932-f10367b1b4df)


### Scouter 수집 Flow
![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/a8441f6d-902e-4faf-bdab-ed3576bd5486)



## 0. 설치 환경 및 설치 버전
- CentOS 7 / windows 11(Scouter window Client) 
- Java 1.8(+ java 11)

### <Pinpoint>  
- Pinpoint v2.3.3
- Hbase 1.4.6
- Flink 1.6.2

### Scouter
- Scouter v2.20.0
- Scouter paper 2.6.4

 
## 1-1. 구조

스카우터는 간단하게 다음과 같은 구조로 이루어진다.
 
![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/774b52a6-a22c-4fee-89aa-956e1711d9ae)

1. Collector(Server) : 
agent들이 측정한 값들을 수집하는 메인 서버를 의미한다.

2. agent.java / agent.host :
측정을 원하는 서버 내부의 tomcat(java.agent)과 환경(java.host)의 정보를 가져온다. 해당 서버에 agent들을 설치하고, Collector Server와 연동시킨다.

3. Client :
Collector가 수집하고 정리한 데이터들을 사용자들이 볼 수 있도록 Client나 paper web의 형태로 보여준다.



## 1-2 : 기본 환경 설정(포트)  
![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/6ae678a9-c8f0-4043-b325-82a8058b6808)

기본적으로 적용해야 할 포트는 아래와 같다. 물론, conf를 통해 변경할 수 있다.

1. 6100 TCP/UDP :
기본적으로 스카우터는 Collector가 agent의 정보를 받기 위해 '6100' 포트를 사용한다.
UDP 포트도 열어주어야 한다.

2. 6188 
Scouter Paper web에서 사용할 기본 연결 주소

## 1-3 : Scouter 설치
 
Scouter 사이트에서 버전에 맞는 스카우터를 받아 압축을 풀어줄 것.

https://github.com/scouter-project/scouter/releases

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/38effe9e-ff64-4d15-a117-76b6ba17b207)

Asset ->  scouter-all-2.20.0.tar.gz를 받아 Linux에서 압축 해제한다.

```sudo tar -xvf '파일'```
 

scouter.client.product-win32 .. 역시 Windows 위에 다운로드한다.  
이는 Windows에서 실행할 클라이언트이다.

다시 리눅스로 돌아가서, 여기서는 편의에 따라 /opt/scouter/ 경로에 설치하였다.

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/a558cb3e-66fa-47d2-9844-2a2a1054a640)


### 1-3-1 Collector 실행
 

/opt/scouter/server의 startup.sh 파일을 실행하여 서버를 가동할 수 있다.

```sudo ./startup.sh```

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/e329ee17-543b-415f-a157-0b62a6a12a52)

단, 가끔 failed to run command `java' 라는 에러가 날 때가 있는데, 이는 java 경로를 인식하지 못해서이다.
이 경우 에디터를 통해 ./startup.sh 파일에서 export로 경로를 지정하여 수정하자.

```bash
#!/usr/bin/env bash


export JAVA_HOME="원래 자바 경로"
export PATH=$JAVA_HOME/bin:$PATH

echo "(테스트용 출력)Java Path : " $JAVA_HOME

nohup java -Xmx1024m -classpath ./scouter-server-boot.jar scouter.boot.Boot ./lib > nohup.out &
sleep 1
tail -100 nohup.out
```

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/e24b47d1-b67a-4a65-b5c1-325c2e8aedcc)


실행 후 nohup log를 확인하여 성공 여부를 파악할 수 있다.  

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/56b72bae-5831-4c17-af88-8fe9aa25cf66)

```ps -ef | grep scouter 로도 확인 가능```  

### 1-3-2 Agent 실행( host )
파악을 원하는 서버에 agent.host와 agent.java 파일을 설치하고, conf 파일을 통해 연결 후 실행하여 Collector와 연동할 수 있다.   
측정하려는 서버 내부의 agent.host 폴더에 들어가서 Conf 파일을 수정해주자.  
![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/bf16bffb-40a8-468d-baea-f983fa68143e)

 ```bash
### scouter host configruation sample
net_collector_ip=Collector 주소
net_collector_udp_port=6100 기본이지만 수정 가능
net_collector_tcp_port=6100
cpu_warning_pct=80
cpu_fatal_pct=85
cpu_check_period_ms=60000
cpu_fatal_history=3
cpu_alert_interval_ms=300000
disk_warning_pct=88
disk_fatal_pct=92
```

이후 agent.host 내부의 host.sh를 실행하여 agent를 실행할 수 있다.  

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/b523c64a-889e-4050-8430-3008bbfac517)

### 1-3-3 Agent 실행( java )
host와 마찬가지로, conf 파일을 수정해야 한다.  

```bash
### scouter java agent configuration sample
obj_name=이름
net_collector_ip=마찬가지로 컬렉터 주소
net_collector_udp_port=6100
net_collector_tcp_port=6100

#hook_method_patterns=sample.mybiz.*Biz.*,sample.service.*Service.*
#trace_http_client_ip_header_key=X-Forwarded-For
#profile_spring_controller_method_parameter_enabled=false
#hook_exception_class_patterns=my.exception.TypedException
#profile_fullstack_hooked_exception_enabled=true
#hook_exception_handler_method_patterns=my.AbstractAPIController.fallbackHandler,my.ApiExceptionLoggingFilter.handleNotFoundErrorResponse
#hook_exception_hanlder_exclude_class_patterns=exception.BizException

## Scouter paper
counter_interaction_enabled=true #Paper 관련 설정
```

java agent는 host처럼 독자적으로 실행하는 것이 아니라, 연관된 Tomcat 실행 시 tomcat이 이 agent(사실 jar 파일이다)를 연관하여 실행하게 동작해야 한다.  

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/f50a7cc2-f16c-4ab6-b951-0a66d1aa4dff)

(scouter.agent.jar 파일이 agent 본체이다)  
측정을 원하는 서버의 tomcat / bin / catalina.sh를 들어가서 스크립트를 추가하자.  

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/475bfc41-c367-4c66-8447-32106f0885e6)

'# ----- Execute The Requested Command -----------------------------------------' 라인(실행 구문) 위쪽에 코드를 집어넣어야 한다.  
혹은 setenv.sh로 집어넣어도 상관없다.  

```bash
#Scouter Agent Setting
SCOUTER_AGENT_DIR="/opt/scouter/agent.java" #agent java 경로(.jar까지 경로가 아님. 베이스 주소 변수이다.)
COLLECTOR_SERVER_IP="192.168.56.1" # Collector 주소
export JAVA_OPTS="$JAVA_OPTS -javaagent:${SCOUTER_AGENT_DIR}/scouter.agent.jar" #사실상 이것이 전체 경로
export JAVA_OPTS="$JAVA_OPTS -Dscouter.config=${SCOUTER_AGENT_DIR}/conf/scouter.conf"
#export JAVA_OPTS="$JAVA_OPTS -Dnet_collector_ip=${COLLECTOR_SERVER_IP}"
```

  
해당 값 저장 후 tomcat을 다시 시작하면 agent jar 파일이 실행될 것이다. grep으로도 확인 가능하다.  

```ps -ef | grep agent.java```  

## 1-4 Scouter windows client
Collector의 저장 값을 Client를 통해서 확인할 수 있다.  
이를 위해 기본적인 연결을 해 줄 것이다.  

### 1-4-1 환경 변수
기본적으로 Scouter client는 Eclipse를 기반으로 만들어졌기에, Java 11 이상의 jdk가 필요하다.  
만약 기본 환경 변수가 1.8이라면, scouter 실행 파일 아래의 scouter.ini에서 다음과 같이 jdk 경로를 따로 지정해줄 수 있다.  


-vm 사용, 위치를 명확하게 집어줄 것.  

```bash
-startup
plugins/org.eclipse.equinox.launcher_1.6.400.v20210924-0641.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_64_1.2.400.v20211117-0650
-data
@user.home/scouter
-vm
C:\Program Files\Java\openlogic-openjdk-11.0.21+9-windows-x64\bin\javaw.exe
-vmargs
-Xms128m
-Xmx1024m
-XX:+UseG1GC
-Dosgi.requiredJavaVersion=11
```

### 1-4-2 클라이언트 로그인 

처음 시작시 아이디와 비밀번호를 입력하라 하는데, admin / admin으로 로그인 가능하다.

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/bf30cb68-6a71-4c75-8600-aa1ff65ae730)



  ![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/84b5ecde-a405-4dd6-9df6-02fdf0f1e000)

로그인 이후 서버의 트랜잭션이 걸릴만한 요청 실행 시 정상적으로 로그가 발생하는 것을 확인할 수 있다.  


## 1-5. Scouter paper web  
공식 메뉴얼이 간단하고 따라하기 쉽다. 다음 내용을 같이 참조하자.  
https://scouter-contrib.github.io/scouter-paper/manual.html  

paper는 Embedded와 StandAlone 방식이 있다.  
이 방식을 주의해서 사용할 것. (다른 정리 내용도 그렇지만, 방식에 따라 설정 및 실행 방법이 다르다.)  
- Paper에서 기본 포트는 6188인데 Scouter의 webapp(API를 제공하는)의 기본 포트이다. (StandAlone)  
- 6180 포트는 scouter에서 webapp을 독립 실행(standalone mode) 하지 않고 scouter collector 서버에 포함(embedded)된 형태로 실행한 경우의 포트이다.  

이 메뉴얼에서는 StandAlone 방식을 사용한다.  
  

### 1-5-1. 환경 설정 webapp
/opt/scouter 내부에 기본적인 파일을 저장했었다.  

webapp에 파일을 풀어주어야 하는데, 이전에 말했던 2가지의 방식이 있어 webapp의 경로가 두 개가 있다.  

/scouter/wepapp과, /scouter/server/wepapp이다. 이번 실행의 경우는 /scouter/wepapp 폴더를 사용하여 부트하였다.  

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/a86ed128-505e-4f30-b77a-1916542f8662)
![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/7ce71adc-952c-4e4d-82c9-934fd81e43a7)

 

extweb 하위에 압축을 푼 paper 파일을 복사한다. index의 경우 이전에도 존재하긴 하나, 덮어써도 상관없다.

 

이후 webapp 하위의 ./startup.sh를 통해 paper를 실행하고, 서버 주소:6188 domain으로 이동 시 정상적으로 paper가 실행되는 것을 확인할 수 있다. 혹은 curl을 통해서 확인도 가능하다.

![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/7f4e1316-edd9-4b6e-b624-f5da2f06d7e8)


만약 java를 찾을 수 없다면, 마찬가지로 이전 케이스와 같이 script 내에 export 경로를 추가하거나 환경 변수를 조절할 것.

 ![image](https://github.com/leon4652/-APM-Pinpoint-Scouter-/assets/93763809/b921aaa0-a116-4462-b680-ff11f0e01fdb)



