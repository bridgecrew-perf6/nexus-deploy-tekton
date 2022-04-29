# nexus-deploy-tekton

## Description
Tekton pipeline을 사용하여 Nexus repository에 존재하는 artifact (jar 파일)의 버전 업데이트 시 서비스 재배포까지의 자동화 환경을 구축하였습니다. 

서비스가 재배포되는 과정들을 Tekton의 Task로 작성하여 Pipeline을 구성하였습니다.

<Pipeline 구성>
- Task_1 : wget 으로 Nexus repository에 존재하는 UPDATED artifact 다운
- Task_2 : kaniko 를 사용하여 image build 및 image registry로 push
- Task_3 : kubectl 명령어를 사용하여, application deploy 

 EventListener 는 Nexus webhook event를 받아 처리하는 api server 역할로서, 들어오는 event를 필터링하여 Trigger Binding과 Trigger Template를 호출합니다.
 
 - Trigger Binding : EventListener로 부터 받은 데이터를 Trigger Template의 파라미터와 매핑
 - Trigger Template : Trigger Binding/EventListener로 부터 어떤 파라미터를 받을 건지, 어떤 파이프라인을 실행시킬 건지 정의 

## 참조 사이트
- Event Triggers : https://tekton.dev/vault/triggers-v0.7.0/
- pipeline 구성 task templates
    - Task_1 : wget [https://hub.tekton.dev/tekton/task/wget]
    - Task_2 : kaniko [https://hub.tekton.dev/tekton/task/kaniko]
    - Task_3 : kubectl-deploy-pod [https://hub.tekton.dev/tekton/task/kubectl-deploy-pod]
- EventListener logs 확인 방법 : https://tekton.dev/vault/triggers-v0.12.1/debugging-eventlisteners/

## 테스트 전 확인 사항
1. Nexus 접속 페이지 확인 및 로그인 진행 
    - NexusURL로 접속하여 로그인 진행
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
                        <url>http://{NEXUS_URL}/repository/{NEXUS_REPOSITORY_NAME}/</url>
                </repository>
        </distributionManagement>
        ```
        
        <~/.m2/settings.xml>
        ```
        <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemalocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
          <servers>
            <server>
              <id>nexus-repo</id>
              <username>{ID}</username>
              <password>{PASSWORD}</password>
            </server>
          </servers>
        </settings>
        ```
3. Nexus webhook 생성
    - Nexus 접속 페이지에서 webhook 생성
    
        ![image](/figure/nexus_page.png)
        
        ![image](/figure/nexus_webhook_1.png)
        
        **{PATH} 는 /ingress/nexus-ingress.yaml 의 {PATH} 와 동일해야 합니다.**
        <br></br>
        ![image](/figure/nexus_webhook.png)

## 테스트 과정
1. apply pv/pvc, role/rolebinding, serviceaccount, ingress
    ```
    kubectl apply -f ./pv-pvc/pipeline-pv.yaml
    kubectl apply -f ./pv-pvc/pipeline-pvc.yaml
    kubectl apply -f ./role-rolebinding/trigger-role-rolebinding.yaml
    kubectl apply -f ./trigger-sa/tekton-trigger-sa.yaml
    kubectl apply -f ./ingress/ingress-nexusdeploy.yaml
    ```
2. apply tasks
    ```
    kubectl apply -f ./task/wget.yaml
    kubectl apply -f ./task/kaniko.yaml
    kubectl apply -f ./task/kubectl-deploy.yaml
    ```
3. apply pipeline
    ```
    kubectl apply -f ./pipeline/pl-nexusdeploy.yaml
    ```
4. apply trigger binding
    ```
    kubectl apply -f ./triggerBinding/tb-nexusdeploy.yaml
    ```
5. apply trigger template
    ```
    kubectl apply -f ./triggerTemplate/tt-nexusdeploy.yaml
    ```
6. apply eventListener
    ```
    kubectl apply -f ./eventlistener/el-nexusdeploy.yaml
    ```
7. apply svc
    ```
    kubectl apply -f ./svc/svc-nexusdeploy.yaml
    ```
## 테스트 실행 결과
1. Eventlistener pod 생성 확인

    ![image](/figure/kubectl_pod_before_deploy.png)

2. Spring 소스코드 mvn deploy 진행
    - Nexus repository 에서 deploy 확인 (http://192.168.9.194:32001/#browse/browse:test-hosted)

3. Pipeline run 동작 확인

    ![image](/figure/kubectl_pod_after_deploy.png)

4. Pipeline 을 통해 배포 된 application 동작 확인

    ![image](/figure/kubectl_check_createdApp.png)
    ![image](/figure/kubectl_logs_echomaven.png)

5. Pipeline 실행 결과
    - Nodeport 확인 (그림에서는 30564)

        ![image](/figure/kubectl_get_svc.png)

    - 웹에서 192.168.9.194:{nodeport}/echo/hello 로 접속

        ![image](/figure/result.png)
