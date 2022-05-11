# Helm

<br>

Kubernetes 패키지 관리를 도와주는 역할을 수행한다.   
Node.js의 npm과 Python의 pip와 같은 역할이라고 보면 된다.

<br>

Helm을 보다 잘 사용하기 위해서는 3가지 주요 개념을 알아야 한다. 
- Chart
- Repository
- Release

<br>

### 1. Chart (차트)

yaml 파일로 정의한 Helm의 패키지로, k8s cluster에서 애플리케이션이 기동되기 위해 필요한 모든 리소스들이 포함되어 있다.

<br>

### 2. Repository (저장소)

차트 저장소로, 차트를 모아두고 공유하는 장소.

<br>

### 3. Release (릴리즈)

k8s cluster에서 구동되는 차트 인스턴스이며 Chart로 배포된 app들은 각각 고유한 버전을 갖고 있다. 일반적으로 동일한 Chart를 여러 번 설치할 수 있고, 이는 새로운 Release로 관리되게 된다. Release될 때 패키지된 차트와 Config가 결합되어 정상 실행되게 된다.

<br>

 <b> 간단한 진행 순서 </b>

- k8s cluster 내부에 Helm Chart를 원하는 Repostiory에서 검색 후 설치
- 각 설치에 따른 새로운 Release 생성

<br>
<br>

## 1. Helm 시작하기
<br>

Helm 설치 ( Kubernetes Cluster가 구성된 환경에서 설치 해야 한다. [kubernetes 설치](https://github.com/HWool/Kubernetes.git) )
```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

<br>

Helm Version 확인
```sh
helm version

version.BuildInfo{Version:"v3.7.0", GitCommit:"eeac83883cb4014fe60267ec6373570374ce770b", GitTreeState:"clean", GoVersion:"go1.16.8"}
```

<br>

## 2. Helm 사용하기
<br>

### 2-1. 차트 Repository 추가 및 업데이트
***
Helm Chart를 사용하기 위해 repository 추가
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/<chart>           # Helm 3
helm install --name my-release bitnami/<chart>    # Helm 2

"bitnami" has been added to your repositories
```

<br>

추가된 repository 확인.
```sh
helm repo list

NAME    URL 
bitnami  https://charts.bitnami.com/bitnami
```

<br>

추가된 repository에 어떤 차트가 있는지 검색
```sh
helm search repo bitnami

NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/bitnami-common                          0.0.9           0.0.9           DEPRECATED Chart with custom templates used in ...
bitnami/airflow                                 12.2.2          2.2.5           Apache Airflow is a tool to express and execute...
bitnami/apache                                  9.0.17          2.4.53          Apache HTTP Server is an open-source HTTP serve...
```

<br>

최신 차트 리스트 사용을 위해 업데이트
```sh
helm repo update
```

<br>

### 2-2. Package 설치하기.
***
`helm innstall` 명령어를 이용하여 패키지를 설치한다.
- Syntax  :   helm install [Release_NAME] [CHART] [flags]

<br>

postgresql을 'db'라는 이름의 namespace에 설치하였고 릴리스 이름은 'postgresql-v1'으로 지정.   
설치 후에는 helm status 명령어로 릴리스의 상태를 확인 가능
```sh
kubectl create namespace db 
helm install postgresql-v1 bitnami/postgresql --namespace db 
helm status postgresql-v1
```
<br>

pod와 service의 상태 확인
```sh
kubectl get pods --namespace db

postgresql-v1-postgresql-0   1/1     Running   0          9m47s
```
```sh
kubectl get services --namespace db

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
postgresql-v1            ClusterIP   10.110.15.51   <none>        5432/TCP   14m
postgresql-v1-headless   ClusterIP   None           <none>        5432/TCP   14m
```
<br>

### 2-3. Upgrade & Rollback
***
새로운 버전의 차트가 릴리스 되거나 구성을 변경할 때는 `helm update` 명령어 사용

```sh
helm upgrade postgresql-v1 stable/postgresql --namespace db 

Release "postgresql-v1" has been upgraded. Happy Helming! NAME: postgresql-v1 LAST DEPLOYED: Sun Sep 12 13:17:37 2021 NAMESPACE: db STATUS: deployed REVISION: 2 TEST SUITE: None NOTES:
```

<br>

반대로 릴리스를 롤백하기 위해서는 `helm rollback` 명령어로를 사용한다. 롤백할 때는 원하는 revision 번호를 찾아서 지정해주면 된다.

```sh
$ helm history postgresql-v1 --namespace db 

REVISION UPDATED                    STATUS        CHART              APP VERSION  DESCRIPTION 
1        Sun Sep 12 12:58:35 2021   superseded    postgresql-8.6.4   11.7.0       Install complete 
2        Sun Sep 12 13:17:37 2021   deployed      postgresql-8.6.4   11.7.0       Upgrade complete

```
```sh
helm rollback postgresql-v1 1 --namespace db 

Rollback was a success! Happy Helming!
```
<br>

### 2-4. Release 삭제하기.
***
릴리스를 삭제하기 위해서는 `helm uninstall` 명령 사용.   
uninstall 실행 시 릴리스는 바로 삭제되기 때문에 Rollback 불가능
```sh
helm uninstall postgresql-v1 --namespace db 

release "postgresql-v1" uninstalled
```

