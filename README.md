# 🚀 Spring Boot 애플리케이션 Kubernetes 배포 

## 🎯 실습 목적
- Spring Boot 애플리케이션을 컨테이너화하는 방법 학습
- Kubernetes 기본 개념 이해 (Deployment, Service, Pod)
- 로컬 개발 환경에서 Kubernetes 클러스터 운영 경험
- 마이크로서비스 아키텍처의 기초 이해
- 컨테이너 기반 애플리케이션의 로깅 및 모니터링 방법 습득


## 🛠 사용 기술

![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Gradle](https://img.shields.io/badge/Gradle-02303A.svg?style=for-the-badge&logo=Gradle&logoColor=white)


## 🏗️ 사전 준비
1. Java JDK 17 설치
   ```bash
   java -version
   ```
2. Docker 설치
   ```bash
   docker --version
   ```
3. Minikube 설치
   ```bash
   minikube version
   ```
4. kubectl 설치
   ```bash
   kubectl version --client
   ```


## 📦 Spring Boot 애플리케이션 준비

1. 프로젝트 구조 확인
   ```
   src
   ├── main
   │   ├── java
   │   │   └── com
   │   │       └── example
   │   │           └── demo
   │   │               ├── DemoApplication.java
   │   │               └── controller
   │   │                   └── TestController.java
   │   └── resources
   │       └── application.properties
   └── test
   ```

2. TestController.java 내용
   ```java
   @RestController
   @Slf4j
   public class TestController {
       @GetMapping("/test")
       public String test() {
           log.info("요청 수락 ~~~");
           return "cicd";
       }
   }
   ```

3. application.properties 설정
   ```properties
   server.port=8999
   ```

4. 애플리케이션 빌드
   ```bash
   ./gradlew build
   ```


## 🐳 Docker 이미지 빌드

1. Dockerfile 생성
   ```dockerfile
   FROM openjdk:17-jdk-slim
   COPY build/libs/demo-0.0.1-SNAPSHOT.jar app.jar
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

2. Docker 이미지 빌드
   ```bash
   docker build -t springapp:v1 .
   ```

3. 이미지 확인
   ```bash
   docker images | grep springapp
   ```


## ☸️ Kubernetes 배포

1. Minikube 시작
   ```bash
   minikube start
   ```

2. Minikube Docker 환경 사용
   ```bash
   eval $(minikube docker-env)
   ```

3. 이미지 다시 빌드 (Minikube 환경에서)
   ```bash
   docker build -t springapp:v1 .
   ```

4. Kubernetes Deployment 생성
   ```bash
   kubectl create deployment springapp --image=springapp:v1 --replicas=3
   ```

5. 배포 상태 확인
   ```bash
   kubectl get deployments
   kubectl get pods
   ```


## 🌐 서비스 노출 및 접근

1. 서비스 생성 (LoadBalancer 타입)
   ```bash
   kubectl expose deployment springapp --type=LoadBalancer --port=8999
   ```

2. 서비스 상태 확인
   ```bash
   kubectl get services
   ```

3. Minikube tunnel 실행 (새 터미널에서)
   ```bash
   minikube tunnel
   ```

4. 서비스 EXTERNAL-IP 확인
   ```bash
   kubectl get services
   ```
   ![image](https://github.com/user-attachments/assets/ec411879-3d0a-48b7-b460-3936aae5cdc1)

   Dashboard에서 확인
   ![image](https://github.com/user-attachments/assets/d10d2b7a-32c7-4a6d-8fc2-2be07d0f4757)


6. 포트포워딩
   ![image](https://github.com/user-attachments/assets/f568a5c8-08a5-43f7-bdc4-4f0e92f0c62f)


7. 브라우저에서 접근
   `http://10.111.60.72:1565/test`
   ![image](https://github.com/user-attachments/assets/13a6af51-8092-4ac8-9248-b3132b472e50)



## 📊 로그 확인 및 모니터링

1. 실시간 로그 확인
   ```bash
   kubectl logs -f -l app=springapp
   ```

2. 특정 키워드로 로그 필터링
   ```bash
   kubectl logs -f -l app=springapp | grep "요청 수락"
   ```
   ![image](https://github.com/user-attachments/assets/3ac60a86-2fad-421e-aa0e-77524e02dd2a)


3. Dashboard 확인
   ![image](https://github.com/user-attachments/assets/52c122c0-7040-4d55-a585-d4e32d8ea57b)


## 🔍 트러블슈팅

### 문제 1: 서비스 접근 불가
- 증상: 브라우저에서 애플리케이션에 접근할 수 없음
- 해결 방법:
  1. 서비스 상태 재확인
     ```bash
     kubectl describe service springapp
     ```
  2. Minikube tunnel 실행 상태 확인
  3. 방화벽 설정 확인
  4. 포트 포워딩 시도

### 문제 2: 404 에러 (Whitelabel Error Page)
- 증상: 브라우저에서 "Whitelabel Error Page" 표시
- 해결 방법:
  1. 컨트롤러 매핑 확인
  2. 애플리케이션 로그 검사
     ```bash
     kubectl logs -l app=springapp
     ```
  3. 컨텍스트 경로 및 포트 설정 확인
  4. 애플리케이션 재시작
     ```bash
     kubectl rollout restart deployment springapp
     ```


## 🧹 정리 및 삭제

1. 서비스 삭제
   ```bash
   kubectl delete service springapp
   ```

2. 디플로이먼트 삭제
   ```bash
   kubectl delete deployment springapp
   ```

3. Minikube 클러스터 삭제
   ```bash
   minikube delete
   ```
   

## 🏗️ 아키텍처 다이어그램

아래는 Spring Boot 애플리케이션이 Kubernetes (Minikube) 환경에서 어떻게 배포되는지를 보여주는 간단한 아키텍처 다이어그램입니다:

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Minikube Cluster                             │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Kubernetes Namespace                       │  │
│  │  ┌─────────────────┐      ┌─────────────────────────────────┐ │  │
│  │  │   Deployment    │      │         Service (LoadBalancer)  │ │  │
│  │  │  ┌───────────┐  │      │                                 │ │  │
│  │  │  │ Pod       │  │      │  ┌─────────────┐                │ │  │
│  │  │  │ ┌───────┐ │  │      │  │ External IP │                │ │  │
│  │  │  │ │SpringApp│◄─┼──────┼──┤   :8999    │◄────────────────┼─┼──┼────┐
│  │  │  │ └───────┘ │  │      │  └─────────────┘                │ │  │    │
│  │  │  └───────────┘  │      │                                 │ │  │    │
│  │  │  ┌───────────┐  │      └─────────────────────────────────┘ │  │    │
│  │  │  │ Pod       │  │                                          │  │    │
│  │  │  │ ┌───────┐ │  │      ┌─────────────────────────────────┐ │  │    │
│  │  │  │ │SpringApp│◄─┼──────┤        Metrics Server           │ │  │    │
│  │  │  │ └───────┘ │  │      └─────────────────────────────────┘ │  │    │
│  │  │  └───────────┘  │                                          │  │    │
│  │  │  ┌───────────┐  │      ┌─────────────────────────────────┐ │  │    │
│  │  │  │ Pod       │  │      │         ConfigMap/Secrets       │ │  │    │
│  │  │  │ ┌───────┐ │  │      │   (Application Configuration)   │ │  │    │
│  │  │  │ │SpringApp│◄─┼──────┤                                 │ │  │    │
│  │  │  │ └───────┘ │  │      └─────────────────────────────────┘ │  │    │
│  │  │  └───────────┘  │                                          │  │    │
│  │  └─────────────────┘                                          │  │    │
│  └───────────────────────────────────────────────────────────────┘  │    │
└─────────────────────────────────────────────────────────────────────┘    │
                                                                           │
                            ┌─────────────────────┐                        │
                            │    Client Access    │                        │
                            │  (Browser/API Call) │◄───────────────────────┘
                            └─────────────────────┘
```

### 구성 요소 설명:

1. **Minikube Cluster**: 로컬 개발 환경에서 Kubernetes를 시뮬레이션합니다.
2. **Kubernetes Namespace**: 리소스들을 논리적으로 구분합니다.
3. **Deployment**: Spring Boot 애플리케이션의 복제본(Pod)들을 관리합니다.
4. **Pod**: Spring Boot 애플리케이션이 실행되는 최소 단위입니다.
5. **Service (LoadBalancer)**: 외부에서 애플리케이션에 접근할 수 있게 해줍니다.
6. **Metrics Server**: 클러스터의 리소스 사용량을 모니터링합니다.
7. **ConfigMap/Secrets**: 애플리케이션의 설정 정보를 저장합니다.
8. **Client Access**: 사용자나 다른 서비스에서 애플리케이션에 접근하는 지점입니다.

이 아키텍처는 기본적인 구성을 보여주며, 실제 프로덕션 환경에서는 추가적인 컴포넌트나 보안 계층이 필요할 수 있습니다.


## 📚 추가 학습 자료
- [Kubernetes 공식 문서](https://kubernetes.io/docs/home/)
- [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)
- [Minikube 튜토리얼](https://minikube.sigs.k8s.io/docs/start/)



이 가이드를 통해 Spring Boot 애플리케이션을 Kubernetes 환경에 성공적으로 배포하고 운영하는 방법을 상세히 익힐 수 있습니다. 실제 환경에서 발생할 수 있는 다양한 상황과 문제 해결 방법도 포함되어 있어, 실무에 바로 적용할 수 있는 지식을 얻을 수 있습니다. 😊
