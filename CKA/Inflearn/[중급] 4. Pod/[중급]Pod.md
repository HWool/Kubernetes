# 1. Pod

## 1-1. Lifecycle

---

인간의 Lifecycle

<img src=./image/pod1-1.PNG>

사람은 태어나서 유아기 -> 청소년 -> 청년 -> 노인 이라는 Lifecycle이 있다.

POD의 Lifecycle

<img src=./image/pod1-2.PNG>

인간의 Lifecyle과 같이 4가지 단계가 있다.
Lifecycle 같은 경우 각 단계에 따라 행해지는 행동이 다르다.

<img src=./image/pod1-3.PNG>

Pod가 있고, `Status` 안에 Pod의 전체 상태를 대표하는 속성인 `Phase`가 있다.
그리고 Pod가 생성되면서 실행하는 단계들이 있는데, 그 단계와 상태를 알려주는 것이 `Conditions`이다. Pod 안에는 Container들이 있고, 그 각각의 컨테이너들의 `State`라고 해서 컨테이너를 대표하는 상태가 있다.

Status에는
Pod를 대표하는 `Phase`가 있고 Pending, Running, Succeeded, Failed, Unknown 5가지가 존재한다.

`Conditions`는 Initialized, ContainerReady, PodScheduled, Ready 4가지가 있고, `Reason`이라고 해서 Conditions의 세부 내용을 알려주는 것도 있다.

ContainerStatuses에는
`State`가 존재하고 Waitinmg, Running, Terminated 3가지가 존재한다.
Conditions와 마찬가지로 `Reason`도 존재한다.

<img src=./image/pod1-4.PNG>

Pod의 최초 상태는 Pending이다.
pod가 어느 노드 위에 올라갈지 결정이 되면 `PodScheduled: True`

InitContainer라고 해서 본 Container가 기동되기 전에 초기화해야 하는 내용이 있을 경우 그 내용을 담는 컨테이너다.
Volume이나 보안 세팅을 위해 사전 설정을 해야 되는 일이 있을 경우
pod생성 내용 안에 initcontainer 항목으로 초기화 스크립트를 넣을 수 있고, 이 스크립트는 본 컨테이너보다 먼저 실행이 돼서 이 작업이 성공적으로 끝났거나, 아예 설정이 되지 않았을 경우엔 `Initialized: True`, 실패하면 `False`를 출력한다.

container에 image를 다운로드하는 동작이 있고,
위에 두 과정동안에는 본 컨테이너는 Wating 상태이며 Reason은 ContainerCreating이다.

Container가 정상적으로 동작을 하게 되면 Pod와 Container 상태는 Running이 된다.
Container가 하나라도 기동 중 문제가 생기면, Container 상태는 Waiting이고, Reason은 CrashLoopBackOff 를 출력한다.

Pod는 Container의 이러한 상태를 running이라고 생각하지만, 대신 내부 ContainerReady와 Ready는 `False`를 출력한다.

Job이나 CronJob으로 생성된 pod의 경우 자신의 일을 수행할 때는 Running,
일을 끝마치면, pod의 상태는 `Faild`, `Succeeded`를 출력한다.
작업을 하고 있는 컨테이너 안에서 하나라도 문제가 생겨 에러가 생기면 `Faild`
작업을 잘 끝마치면 `Succeeded`

이때 Faild든, Succeded든 Pod 내부 ContainerReady와 Ready는 `False`를 출력한다.

추가적인 case로 `Pending` 중에 pod 상태가 `Faild`로 될 수도 있고,
`Pending`이나 `Running` 상태일 때 통신 상태가 불안정하면 `Unknwon` 상태가 되는데,
통신 장애가 원할하게 해결 되면 정상적으로 돌아가지만, 장애가 지속되면 `Faild`로 변경된다.

# 2. ReadinessProb, LivenessProbe

## 2-1. 사용 필요성

---

<img src=./image/pod2-1.PNG>

pod를 만들면 컨테이너가 생기고, running 상태가 되면서 app이 잘 동작을 하게 될 것이다.
service와 연결을 하게 되면 외부에서 실시간으로 app에 접근을 할 수 있을 것이다.

1. 위 사진에서 node1과 node2만 연결되고, 각각 트래픽이 50%씩 간다고 가정을 하게 되면,
   node2 서버가 down이 되면 pod1으로 100% 갈 것이다. pod2는 autohealing으로 인해 node3에 다시 생성이 되는데, running 상태여도 app은 booting일 수도 있다.
   이때, 트래픽은 다시 pod2로 가게되고, 50%의 고객들은 app을 사용할 수 없을 것이다.

-> 위와 같은 상황을 만들지 않기 위해 ReadinessProbe를 설정해줘야 한다.
ReadinessProbe가 app이 구동되기 전까지 서비스와 연결을 시키지 않기 때문이다.

2. 이러한 상태에서 app만 down이 된다면? (pod와 container는 running 상태)
   다시 50% 고객은 app을 사용할 수가 없게 된다.

-> 이때 app에 대한 장애상황을 감지해주는게 LivenessProbe이다.
pod를 만들 때 LivenessProbe를 주게 되면 해당 에러가 생기면 파드를 재시작해서 잠깐의 트래픽 에러는 발생하겠지만 지속적으로 에러 상태가 되는 것을 방지해준다.

## 2-2. 사용 방법

---

<img src=./image/pod2-2.PNG>

ReadinessProb, LivenessProbe는 사용 목적이 틀를 뿐 설정은 비슷하다.
크게 `httpGet`, `Exec`, `tcpSocket`으로 app에 대한 상태를 확인한다.

`httpGet`은 port, host 이름, path는 httpHeader, Scheme를 확인할 수 있다.
`Exec`은 특정 명령어를 날려서 그에 따른 결과를 확인하고,
`tcpSocket`은 port번호와 host명을 확인해서 ReadinessProb, LivenessProbe에 대한 성공여부를 확인할 수 있다.

-> 이 3가지 옵션은 하나라도 있어야 한다.

옵션 값들도 있는데
initailDelaySeconds는 최초 Probe를 하기 전에 delay 시간
periodSeconds는 Probe를 체크하는 시간의 간격
timeoutSeconds는 지정된 시간까지 결과가 와야한다.
sucessThreshold는 몇 번 성공 결과를 받아야 정말로 성공으로 인정을 할 것인지
failureThreshold는 몇 번 실패 결과를 받아야 정말로 실패로 인정을 할 것인지에 대한 값이다.

<img src=./image/pod2-3.PNG>

한 서비스의 파드가 연결되어 있는 상태에서 파드를 하나 더 만들 것인데, 컨테이너의 hostpath로 node의 볼륨이 연결되어 있다. 이 컨테이너에 ReadinessProb를 설정을 할 것임.

`Exec`를 통해 `cat /readiness/ready.txt` command를 날릴 것이고, 옵션들을 설정 해줄거임.

이렇게 ReadinessProbe를 설정해주고 Pod를 생성하면, pod와 container는 running 상태이지만, ReadinessProbe가 성공을 하기전까지는 ContainerReady와 Ready는 `False`를 출력한다.

이런한 False 상태가 지속되면 Service의 Endpoint는 `NotReadyAddr`로 생각하고 연결을 하지 않는다,

<img src=./image/pod2-4.PNG>

한 서비스의 두 파드가 동작 중임.
k8s는 /health가 있는지 확인을 할 것이다. 만약 3회동안 실패를 하게된다면 k8s 문제가 있다고 생각하고 pod를 restart를 하게 된다.

# 3. QoS classes

## 사용 필요성

---

<img src=./image/pod3-1.PNG>

1. Node에 resource가 있고, pod들이 이 자원을 균등하게 사용하고 있을 때,
   pod1에서 더 많은 자원이 필요한 상황이 발생했다. 하지만 node에는 자원양이 남아 있지 않는데, 이러면 pod1은 자원 부족으로 down이 되어야 할까?
   아니면 pod2나 pod3를 다운시키고 자원을 할당시켜줘야 할까?

k8s는 app의 중요도에 따라 3가지 단계로 QoS를 제공해준다.

위와 같은 상황에서는 BestEffot가 부여된 pod가 먼저 down이 되고, pod1에게 자원을 할당해줄 것이다.

2. node에 자원이 남아 있지만 pod2에서 그보다 많은 자원을 필요로 한다면,
   `Burstable`이 부여된 pod가 down이 된다. Guranteed가 가장 마지막까지 파드를 안정적으로 유지 시켜주는 class이다.

QoS는 특정 속성이 있어서 설정을 하는 것은 아니고, container의 resource설정 부문에서 requests와 limit 에 따라서 mem과 cpu를 어떻게 설정하는지에 따라 결정이 된다.

## 3-1. (Guaranteed, Burstable, BestEffot)

---

<img src=./image/pod3-2.PNG>

1. Guaranteed는 pod에 여러 컨테이너가 있다면 모두 request와 limit가 있어야하고, mem과 cpu가 설정이 되어야 한다. 그리고 그 값들은 같아야 한다.

2. Burstables는 container마다 request와 limit가 모두 설정되어 있어도, mem과 cpu의 값이 다르거나, limit는 설정이 되어 있지 않는경우, 또는 한 컨테이너는 완벽하게 설정되어 있지만, 다른 컨테이너는 설정이 되어 있지 않는 경우 등 다양하다.

3. BestEffot는 pod의 어떤 request와 limit가 모두 미설정되어 있어야 한다.

-> 3 가지 클래스는 request와 limit를 어떻게 설정하느냐에 따라서 결정이 되는 것이다.

<img src=./image/pod3-3.PNG>

Burstables 같은 경우에는 여러 컨테이너가 있을 때 어떤 컨테이너가 먼저 삭제되는지에 대해서도 알 필요가 있다.

이거는 OOM Score에 따라서 결정이 된다.  
두 개의 컨테이너 app이 사용하는 mem이 4G인데, Request가 각각 5G, 8G가 라면
사용량이 75%, 50%로 볼 수 있다.
이때 K8S는 사용량이 높은 POD부터 먼저 제거한다.

# 4. Node Scheduling

<img src=./image/pod4-1.PNG>
K8S에서 POD를 어떤 노드에 할당할지 자동으로 정해줄 수 있고, 때로는 직접 정할 수도 있다.

K8S에서 직접 노드 설정하는 방법.
NodeName, NodeSelector, NodeAffinity는 pod를 특정 노드에 할당되도록 선택하기 위한 용도.

node들이 있고, 모두 사용할 수 있는 CPU양이 정해져 있고, Label이 달려 있음.
node1~3은 서버가 한국에 있고, 4~5는 미국에 있다고 가정.

이제 pod를 하나 생성하면, k8s 스케쥴러는 cpu자원이 가장 많은 노드에 파드를 할당 시켜주는데,
첫째로 NodeName을 사용하면, 스케쥴러와 상관없이 명시한 노드로 할당할 수 있다.
-> 실제 사용환경에선 노드 생성 및 삭제가 일어나 이름이 변경될 수 있기 때문에 잘 사용안한다.

두번째로 NodeSelector를 실제 환경에서 자주 쓰는데, Pod에 key와 value를 달면 해당 label이 달려있는 node에 할당된다.
-> pod에서 지정한 label과 일치하는 노드과 없다면 생성되지 않는 오류가 발생

세번째로 NodeAffinity는 위 NodeSelector의 단점을 보완해서 사용할 수 있다.
pod에 key만 설정을 해도 해당 그룹에 자원이 많은 node에 할당이 되고, key가 맞지 않더라도, 스케쥴러가 판단해서 자원이 많은 노드에 할당이 되도록 옵션을 줄 수 있다.

Pod간 집중/ 분산
여러 파드들을 한 노드에 집중해서 할당을 하거나, 파드들 간에 겹치는 노드 없이 분산을 해서 할당할 수 있다.

웹과 서버 파드가 있고, 한 hostpath를 사용하느 pv에 연결이 되어 있으면, 웹과 서버 파드는 한 노드에 위치해 있어야 한다. 그래서 두 파드를 한 노드에 할당하려면 Pod Affinity를 사용해야 한다.

웹 파드가 스케쥴러에 의해 특정 한 노드에 할당이 되면, 노드애 pv가 생성이 될 것이다. 이후 서버 파드 label을 웹 서버의 라벨과 동일시 해주면 자연스럽게 같은 노드에 생성이 될 것이다.

master가 죽으면 slave가 백업을 해줘야 하는 관계인데, 두 파드가 같은 노드에 들어가게 되면, 노드가 다운 시 master, slave 둘다 다운이 되기 때문에 분산시켜줘야 한다.

master가 스케쥴러에 의해 한 노드에 들어가고, slave 파드를 만들 때 Anti-Affinity를 줘서 master pod의 key value를 설정을 해주면 이 파드는 master와 다른 노드에 할당이 된다.

Node에 할당제한.
특정 노드에는 아무 pod나 할당이 되지 않도록, 제한을 하기 위해 사용을 한다.
Node5는 높은 사양의 그래픽을 필요로 하는 웹을 돌리기 위한 GPU로 설정르 해놓았을 때, 운영자는 taint라는 것을 설정한다.
일반적인 pod들은 스케쥴러가 node5로 지정해주지 않고, nameselector로 지정해도 할당되지 않는다. 이 노드에 할당이 될려면 파드에 Toleration을 달고 생성해야 한다.

## 4-1. NodeAffinity

---

<img src=./image/pod4-2.PNG>
<img src=./image/pod4-3.PNG>

`matchExpressions`라는 속성을 통해서 pod는 node들을 선택할 수 있는데, key를 그룹핑 단위로 그리고 파드를 하위 식별자로 붙어진 라벨들이 노드에 붙여져 있고, key가 kr인 그룹안에 할당을 하고 싶을 때 matchExpressions를 사용할 수 있다.

`required, preferred`라는 속성도 있는데,
required 속성을 가진 파드가 있다면, 이 속성의 값이 있는 노드가 있지 않다면, 할당되지 않는다.
하지만 preferred 속성을 가진 파드가 있다면, 이 속성의 값을 선호할 뿐이고 없더라도 노드에 할당시켜준다.

<img src=./image/pod4-4.PNG>
preferred 속성에는 `weight`라는 필수 속성이 있는데, key가 다른 node가 두 개가 있고, cpu의 값도 다르다. 
하나의 파드를 생성하는데, preferred의 속성 값이 두 개가 있는데, 두 개 노드 모두 일치해, cpu가 높은 node1으로 할당해주는 게 맞지만, `weight`라는 옵션을 통해 달라질 수 있다.
key가 us인 node weight는 10이고, kr인 node weight는 50인데, 스케쥴러는 최초 노드1에 50 노드 2에 30의 점수를 주었지만, weight의 값을 더해서 노드 1은 60, 노드 2은 80으로 최종적으로 노드2에 할당이 된다.
(위와 같이 점수를 주지는 않지만 대략적으로 보여주기 위한 것.)

## 4-2. Pod Affinity

---

- Node Affinity와 같이 required와 preferred 속성을 쓸 수 있음

<img src=./image/pod4-5.PNG>

- type:app이라는 라벨을 가진 web 파드가 node1에 할당이 됨.
- server pod는 PodAffinity를 통해 matchExprssions로 node가 아닌 pod에 매칭되는 label을 찾도록 함. -> web pod와 같이 노드1에 할당이 될 수 있음.
- topologyKey는 Node의 key 보기 때문에, 같은 key를 가진 노드에서만 찾겠다는 의미
- 만약 web 파드가 node3에 있었다면 server pod는 pending 상태에 있다가 오류가 날 것이다.

## 4-3. Pod Anti-Affinity

---

<img src=./image/pod4-6.PNG>

- master pod가 node4에 할당이 됨.
- slave pod에는 podAntiAffinity에다 master1의 라벨을 달면, master1이 할당된 node와는 다른 노드로 할당시켜준다.

## 4-4. Taint, Toleration

---

<img src=./image/pod4-7.PNG>

- node1은 gpu가 구성되어 있음.
- 일반적인 pod들이 node1으로 할당되지 않기 위해 Taint를 설정
- label(hw : gpq), effect : NoSchedule -> 다른 pod들이 스케쥴링 되지 않음.
- pod에 Toleration을 달면 달 수 있는데, key와 value값, operator, effect 값까지 모두 node의 Taint 값과 같아야지만 할당이 가능함
- 하지만 pod가 생성될 때 다른 노드로 생성될 수 있기 때문에, node에 label을 달아 nodeSelector와 같이 지정해줘야 한다.

- effect가 NoExecute일 때 Pod Toleration에 tolerationSeconds 옵션을 달 수가 있는데, 이것을 달게 되면, 지정 시간 이후에는 삭제가 된다는 의미이다.

---

<b>effect</b>

- NoSchedule : 다른 pod들이 스케쥴링 되지 않게 해줌
- PreferNoSchdule : 가급적 스케쥴링이 되지 않게 해주는건데, 다른 노드들이 여유가 있지않으면 파드가 할당 되게 해준다.
- NoExecute : 파드가 있는 기존 노드에 이 옵션을 달면 파드는 사라진다.
  (! 다른 옵션들은 기존 노드에 달아도 파드들이 사라지지 않음)

<b>offer</b>

- Equal
- Exists

---
