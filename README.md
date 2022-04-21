# nexus-deploy-tekton

## 참조 사이트
- Event Triggers [https://tekton.dev/vault/triggers-v0.7.0/]
- Pipeline 구성 task templates
    - Task_1 : wget [https://hub.tekton.dev/tekton/task/wget]
    - Task_2 : kaniko [https://hub.tekton.dev/tekton/task/kaniko]
    - Task_3 : kubectl-deploy-pod [https://hub.tekton.dev/tekton/task/kubectl-deploy-pod]
- EventListener logs 확인 방법 [https://tekton.dev/vault/triggers-v0.12.1/debugging-eventlisteners/]

## 테스트 전 확인 사항
1. Nexus 접속 페이지 확인 및 로그인 진행 
    - http://192.168.9.194:32001 에 접속하여 로그인 진행 (아이디: admin / 비밀번호: admin)
    
2. Spring 소스코드 준비
    - Sample spring code
        ```
        git clone https://github.com/sihyunglee823/nexus-deploy-tekton-sourceCode.git
        ```
    - 위의 sample spring code 사용하지 않을시, pom.xml 과 settings.xml 파일에 nexus 계정정보를 추가하여야 합니다.
    
        <pom.xml>
        ```
         <distributionManagement>
                <repository>
                        <id>nexus-repo</id>
                        <name>nexus-repo</name>
                        <url>http://192.168.9.194:32001/repository/test-hosted/</url>
                </repository>
        </distributionManagement>
        ```
        
        <~/.m2/settings.xml>
        ```
        <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemalocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
          <servers>
            <server>
              <id>nexus-repo</id>
              <username>admin</username>
              <password>admin</password>
            </server>
          </servers>
        </settings>
        ```
3. Nexus webhook 설정 확인
    - Nexus 접속 페이지에서 webhook이 등록되어있는지 확인
    
        ![image](/figure/nexus_page.png)
    - 해당 webhook 클릭 후 settings 이 그림과 같이 되어있는지 확인 (repository 이름, component, URl 확인)

        ![image](/figure/nexus_webhook.png)


## 테스트 과정
1. apply tasks

    ```
    kubectl apply -f ./task/wget.yaml
    kubectl apply -f ./task/kaniko.yaml
    kubectl apply -f ./task/kubectl-deploy.yaml
    ```
2. apply pipeline
    ```
    kubectl apply -f ./pipeline/pl-nexusDeploy.yaml
    ```
3. apply trigger binding
    ```
    kubectl apply -f ./triggerBinding/tb-nexusDeploy.yaml
    ```
4. apply trigger template
    ```
    kubectl apply -f ./triggerTemplate/tt-nexusDeploy.yaml
    ```
5. apply eventListener
    ```
    kubectl apply -f ./eventlistener/el-nexusDeploy.yaml
    ```
6. apply svc
    ```
    kubectl apply -f ./svc/svc-nexusDeploy.yaml
    ```
## 테스트 실행 결과
