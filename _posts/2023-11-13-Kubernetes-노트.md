---
title: Kubernetes 노트
author: chromato99
date: 2023-11-13 23:00:00 +0900
media_subpath: /assets/img/posts/2023-11-13-Kubernetes-노트/
categories: [Cloud Native, K8s]
tags: [linux, k8s, cloudnative]
---

## Kubernetes 노트

지난 가천 카카오엔터프라이즈 SW 아카데미 기간동안 프로젝트를 진행하면서 노션에 작성한 Kubernetes 노트 내용을 옮긴 것이다.

Team Github: <https://github.com/KEA-ACCELER><br>
A-Log(동시편집 기반 릴리즈 노트 공유 시스템) : <https://github.com/KEA-ACCELER/alog-cluster><br>
A-Form(GPT 기반 설문 플랫폼) : <https://github.com/KEA-ACCELER/aform-cluster>

## 0. 참고자료

쿠버네티스란? 컨테이너 오케스트레이션 플랫폼

[Overview](https://kubernetes.io/docs/concepts/overview/)

참고할 만한 한국어 페이지

[쿠버네티스(kubernetes) (1) - 기본 이론 및 설치 구성](https://hoing.io/archives/52)

[[번역]쿠버네티스 패킷의 삶 - #1](https://coffeewhale.com/packet-network1)

[[Kubernetes] 쿠버네티스 설치(kubeadm) 및 cluster 구성하기](https://kindloveit.tistory.com/23)

### 2023 쿠버네티스 표준 아키텍처

[2023년 쿠버네티스 표준 아키텍처 &#124; 요즘IT](https://yozm.wishket.com/magazine/detail/1998/)

쿠버네티스용 기술 스택을 참고하기 좋은 링크

### 쿠버네티스 구성요소

Kubernetes components

[Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)

![kubernetes-cluster](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

쿠버네티스는 기본적으로 control plane이라 불리는 마스터 노드와 여러 컨테이너들이 실행되는 worker node로 구성되어 있다.

모든 노드에는 기본적으로 쿠버네티스가 똑같이 설치 되어 있어야 마스터 노드아래에 워커노드가 붙어 동작하도록 구성된다.

마스터 노드는 kube-apiserver, etcd, kube-scheduler, kube-controller-manager, 등으로 구성되고 이들도 각각 컨테이너로 동작하는것 같다.

워커노드는 pod이라는 형태의 컨테이너 그룹을 host 하도록 되어 있다.

#### Node

각각의 노드는 logical 또는 physical 컴퓨터를 의미한다. 

[Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/)

#### Pod

Pod는 Kubernetes에서 만들고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위이다.

쿠버네티스에서는 pod당 하나의 IP가 부여된다고 한다.

[Pods](https://kubernetes.io/docs/concepts/workloads/pods/)

#### Deployment

배포는 Pod 및 ReplicaSet에 대한 선언적 업데이트를 제공한다. pod이 deployment로 배포된 컴퓨팅 단위라면 deployment는 그 단위들을 어떻게 배포할지 선언하는 부분인것 같다. 선언하면 deployment controller가 선언에 따라 pod을 생성하거나 삭제한다고 한다.

#### Service

Kubernetes에서 서비스는 클러스터에서 하나 이상의 포드로 실행 중인 네트워크 애플리케이션을 노출하는 방법이다.

이는 컨테이너를 실행하는 부분과 네트워킹 부분을 분리하여 관리하기 위함이다.

[Service](https://kubernetes.io/docs/concepts/services-networking/service/)

![kubernetes-service](/kubernetes-service.png)

## 1. 쿠버네티스 설치

쿠버네티스를 설치하기 위해서는 몇가지 고려해야 할 것이 있다. 아래에 설치 방법들은 첨부한 링크에서 자세히 확인할 수 있다.

[Getting started](https://kubernetes.io/docs/setup/)

1. 쿠버네티스 다운로드
    
    원래는 쿠버네티스 구성요소를 직접 다운로드 할 수 있음
    
    하지만 직접 하나하나 설정하기엔 귀찮으므로 쿠버네티스 클러스터를 구성해주는 솔루션 툴을 사용함 (따라서 쿠버네티스 다운로드는 패스)
    
2. Tools 설치
    
    [Install Tools](https://kubernetes.io/docs/tasks/tools/)
    
    kubectl은 커맨드라인 툴이므로 필수이고
    
    클러스터 구성 솔루션은 대표적으로 minikube와 kubeadm이 있다.
    
    minikube는 싱글 노드용 쿠버네티스로 간단하게 사용할 수 있어 학습이나 연구용으로 많이 사용된다.
    
    [minikube start](https://minikube.sigs.k8s.io/docs/start/)
    
    kubeadm은 프로덕션 환경에서 멀티노드를 사용하기 위해 사용 (훨씬 복잡함)
    
    A-Form 프로젝트 구축에는 kubeadm을 사용하였다.
    
    [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
    
3. Container runtime 설치
    
    쿠버네티스는 컨테이너 오케스트레이션 플랫폼인 만큼 컨테이너 런타임을 설치해 주어야 한다.
    
    최근 도커를 의존하던 것을 제거하고 containerd라는 표준 컨테이너 규격을 사용하는것으로 변경되었다.
    
    도커 또한 containerd기반으로 변경되었으므로 도커를 설치했다면 containerd 또한 같이 설치된다.
    
    하지만 몇가지 기본 설정이 다를 수 있어 아래 링크를 참고해 설정을 해주어야 할 수 있다.
    
    [Container Runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
    

여기까지 설정하면 기본적인 설치는 완료했다 볼 수 있다.

## 2. 쿠버네티스 설정 및 실행

### kubernetes tutorial (with minikube)

[Tutorials](https://kubernetes.io/docs/tutorials/)

### kubernetes with kubeadm

준비사항:

방화벽 사용시 쿠버네티스용 포트를 열어줘야함

**control plane endpoint 설정**

control plane endpoint는 control plane의 endpoint 즉 도메인 이름을 설정하는 부분이다.

이를 aws 같은 클라우드에서 구축하는 것이라면 dns 서비스 등을 사용하면 되지만 그게 아니라면 로컬 dns를 설정해 주어야 한다.

간단하게 설정해주기위해 모든 노드의 /etc/hosts 파일에 endpoint를 설정해주었다.

```
<control plane node ip> cluster-endpoint
```

그런다음 kubeadm init 명령으로 control plane 초기화를 해주었다.

```bash
kubeadm init --control-plane-endpoint=cluster-endpoint \
--pod-network-cidr=192.168.0.0/16 \
--apiserver-advertise-address=172.16.211.25 \
--ignore-preflight-errors=NumCPU
```

[Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

--control-plane-endpoint : control-plane의 공유 endpoint를 설정. control-plane을 싱글이 아닌 복수의 노드로 설정해 availability를 높일때 필요함 (후에 수정 안됨)

control-plane endpoint는 다른 노드에서도 설정한 이름으로 찾을 수 있어야 하므로 DNS를 설정하거나 /etc/hosts에 주소를 설정해야함

--apiserver-advertise-address=172.16.211.64 : control-plane 노드의 ip를 외부에 공개하기위한 옵션?

--pod-network-cidr=192.168.0.0/16 : calico 사용을 위한 네트워크 대역 설정(사용하지 않는 대역으로 설정해야함)

--ignore-preflight-errors=NumCPU : CPU 최소 개수 제한을 무시하기 위한 옵션

kubeadm init이 성공적으로 이루어지면 안내에 따라 몇가지 설정을 해주어야한다.

KUBECONFIG 환경변수를 설정하고(환경변수 설정 제대로 안되면 kubectl 동작 안됨) networking addon들을 설정해주어야 함

### Networking and Network Policy

[https://kubernetes.io/docs/concepts/cluster-administration/addons/](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

쿠버네티스를 위한 네트워킹 솔루션은 여러가지가 있다.

이중에 하나를 고르면 되는데 Cilium, Calico, Funnel이 유명하다는것 같다.

#### Calico

*NetworkManager 서비스를 사용할 경우 아래 설정을 해주어야 할 수 있다.

[Troubleshooting and diagnostics &#124; Calico Documentation](https://docs.tigera.io/calico/latest/operations/troubleshoot/troubleshooting#configure-networkmanager)

Calico 설치는 아래 문서를 참고하였다.

[Install Calico networking and network policy for on-premises deployments &#124; Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico)

[Install using Helm &#124; Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/helm)

> 3.26버전엔 문제가 있어 3.25.1로 설치해 주었다. (후에 Cilium으로 교체)
{: .prompt-info }

[https://github.com/projectcalico/calico/issues/7715](https://github.com/projectcalico/calico/issues/7715)

쿠버네티스 네트워킹에 대한 문서도 읽어볼만 하다.

[About Kubernetes Networking &#124; Calico Documentation](https://docs.tigera.io/calico/latest/about/about-k8s-networking)

### Cilium

[Cilium - Cloud Native, eBPF-based Networking, Observability, and Security](https://cilium.io/)

[Cilium Quick Installation — Cilium 1.13.4 documentation](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#k8s-install-quick)

Cilium은 CNCF에서 incubating하는 CNI 이다. CNCF 프로젝트인 만큼 Cilium을 사용해 주었다.

Cilium CLI를 설치한후 cilium install 명령을 사용해 설치할 수 있다.

### worker node join

이제 worker node를 추가해 주어야 한다. 추가를 위한 명령어는 init이 성공적으로 이루어지면 출력 되는걸 확인 할 수 있다.

이때 token을 사용해야 하는데 토큰 리스트는 kubeadm token list 명령으로 확인할 수 있다.

```bash
kubeadm join cluster-endpoint:6443 --token mxskr3.cgddhlfck5q1ms2f --discovery-token-ca-cert-hash sha256:44d05a799345bd79f836ecf1cc29cb5f03bf0a3fb3ae81b8025cea480a4315c0
```

> 토큰은 기본적으로 24시간 후에 소멸하므로 그 이후에는 새로 생성해 주어야 한다
{: .prompt-info }

```bash
kubeadm token create
```

node 상태는 kubectl get nodes로 확인할 수 있다.

![kubectl-get-nodes](/kubectl-get-nodes.png)

#### **Single machine에서 kubeadm을 사용하고 싶을 경우

[Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#control-plane-node-isolation)

```bash
kubectl taint nodes --all [node-role.kubernetes.io/control-plane-](http://node-role.kubernetes.io/control-plane-)
```

위의 명령으로 control-plane 노드에 pod을 생성할 수 있도록 해주어야 한다.

### Helm 설치

2023.05.14 추가: 쿠버네티스를 위한 여러 도구들을 설치하다보니 이를 편하게 설치하고 관리할 수 있으면 좋을것 같다 생각했는데, Helm이라는 쿠버네티스용 관리 도구가 있다는것을 알고 내용 추가

#### Helm 이란?

[Helm &#124; Using Helm](https://helm.sh/docs/intro/using_helm/)

Helm은 쿠버네티스를 위한 패키지 관리 도구이다. Kubernetes의 오브젝트들의 구성체인 YAML을 패키지 형태로 관리한다. Helm 패키지는 YAML 형식으로 구성되어 있으며, 이것을 chart라고 한다.

#### Helm 사용법

[Helm &#124; Using Helm](https://helm.sh/docs/intro/using_helm/)

## 3. Kubernetes 서비스 구축

### Kubernetes NGiNX Ingress Controller 설치

[https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)

이제 kubernetes 클러스터 설정이 끝났으니 본격적으로 서비스를 구성하고 외부에서 접근할 수 있도록 해야한다. kubernetes를 사용한 만큼 여러 서비스를 한번에 배포할건데 이들 서비스로의 연결을 어떻게 해줄지 고민하고 nginx ingress controller가 있다는것을 알게 되었고 이를 사용하기로 했다.

설치를 비롯한 여러 설정은 위의 링크에서 참고 할 수 있다.

여기서는 우선 on-premise 서버에 설치하는 것이므로 bare metal을 기준으로 설치하였다.

[Installation Guide - Ingress-Nginx Controller](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters)

```bash
# Using helm
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
	-f nginx-values.yml \
  --namespace ingress-nginx --create-namespace

# Using manifest
kubectl apply -f [https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/baremetal/deploy.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/baremetal/deploy.yaml)
```

위 명령을 입력하면 설정파일을 기반으로 자동으로 nginx ingress controller를 설치하고 배포해준다.

다만 그전에 고려해야할 설정이 있다.

#### Kubernetes NGiNX Ingress Controller 설정

[Bare-metal considerations - Ingress-Nginx Controller](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)

aws와 같은 cloud 환경이라면 클라우드에서 제공하는 load balancer를 사용해 kubernetes와 연동하여 진입점을 구성해주면 되지만 지금 구성하는 환경은 bare metal 이므로 구성에 대한 고려가 필요하다.

위의 링크가 baremetal 일 경우의 설정에 대한 guide인데 구현의 심플함을 위해 NodePort 방식으로 네트워크를 설정해 주기로 했다.

![ingress-nginx-nodeport](https://kubernetes.github.io/ingress-nginx/images/baremetal/nodeport.jpg)

NodePort가 무엇인지 참고 : [https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/) 

하지만 NodePort를 사용하면 아무 노드에서나 접근할경우 pod이 있는 노드를 찾아 hop 해야하는 문제가 있으므로 nginx-ingress-controller를 배포할 node를 추가적으로 생성하고 해당 노드를 gateway처럼 사용하는 방식으로 설정하는 것이 좋다 생각했다.

이를 위해 node들에 label을 부여하고 nodeSelector 옵션을 사용해 서비스들의 배포 node를 설정해 주었다.

```bash
# Node에 라벨 부여
kubectl label nodes <node name> key=value
```

```bash
# Helm values 파일 수정
helm show values ingress-nginx/ingress-nginx > nginx-ingress.yml

# values 파일을 사용해 ingress-nginx 설치 또는 이미 배포된 helm charts에 values 적용
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
	-f nginx-values.yml \
  --namespace ingress-nginx --create-namespace

# 만약 helm이 아닌 manifest로 설치했을 시
kubectl patch deployment <deployment name> -p '{"spec":{"template":{"spec":{"nodeSelector":{"type":"worker"}}}}}’
```

그런다음 ingress config 파일을 작성해주어 그것을 배포해주면 된다.

ingress의 경우 아래와 같이 구성해 주었다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: aform-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: server.acceler.kr
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: aform-ws
                port:
                  number: 80
          - path: /app/user
            pathType: Prefix
            backend:
              service:
                name: aform-was-user
                port:
                  number: 8080
          - path: /app/survey
            pathType: Prefix
            backend:
              service:
                name: aform-was-survey
                port:
                  number: 8080
          - path: /app/ai
            pathType: Prefix
            backend:
              service:
                name: aform-was-ai
                port:
                  number: 8080
```

![nodes-overview](/nodes-overview.png)

이렇게 구성한 노드 구조를 그림으로 나타내면 위와 같다고 할 수 있다.

Gateway Node에 nginx ingress controller를 배포하고 이를 집입점으로 사용한다. 그러면 ingress controller가 url의 path를 기반으로 적절한 서비스로 연결을 해주는 방식이다.

추가적으로 gateway용으로 사용하는 노드로의 접근 외의 다른 접근은 drop하기 위해 externalTrafficPolicy를 Local로 설정한다.

```bash
# 이미 배포된 서비스 수정 방법
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec":{"externalTrafficPolicy":"Local"}}’
```

### Taints and Tolerations

위의 ingress controller 설정에서는 노드에 라벨을 부여하는 방식으로 노드를 선택하였는데, 이는 pod이 생성될때 특정 노드를 선택하는 방법이다.

이와 다르게 taints와 tolerations라는 기능을 사용하면 기본적으로 pod이 생성되지 못하도록 하고 허가된 노드만 특정 노드에 pod을 생성하도록 할 수도 있다. 이를 통해 하나의 노드를 ingress controller 전용으로 설정할 수 있고, 현업에서는 GPU가 있는 노드의 배포 설정을 하는 용도 등에 사용한다.

```bash
# 특정 노드에 taint를 추가하는 명령
kubectl taint nodes node-name key1=value1:NoSchedule

# 이미 배포된 deployment에 toleration을 patch하는 명령
kubectl patch deployment deployment-name --type=json -p '[{"op": "add", "path": "/spec/template/spec/tolerations", "value": [{"key": "key1", "operator": "Exists", "effect": "NoSchedule"}]}]'

# 이미 배포된 pod에 toleration을 patch하는 명령
kubectl patch pod pod-name --type=json -p '[{"op": "add", "path": "/spec/tolerations", "value": [{"key": "key1", "operator": "Exists", "effect": "NoSchedule"}]}]'
```

[Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

### Kubernetes 로컬 registry 설정 (Optional)

Kubernetes는 여러 노드에서 돌아가기때문에 단순히 한 노드에서 docker로 이미지를 빌드하고 이를 사용해 배포를 하려하면 imagePullErr가 발생하게 된다. 따라서 docker hub와 같은 공개된 저장소를 사용하거나 private registry를 구축해서 사용해야 한다. 

검색 결과 도커 registry 라는 로컬 registry 구축을 위한 이미지를 찾을 수 있었고 이를 사용해 kubernetes cluster 상에 private registry를 구축해 사용하는 방법을 선택했다.

[Deploy a registry server](https://docs.docker.com/registry/deploying/)

[registry - Official Image &#124; Docker Hub](https://hub.docker.com/_/registry)

cluster 상에 deploy하기 위해 아래와 같이 yaml 파일을 작성해 주었다.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: docker-repo-pv
spec:
  storageClassName: docker-registry
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/repository
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: docker-repo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi 
  storageClassName: docker-registry
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry
  labels:
    app: docker-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-registry
  template:
    metadata:
      labels:
        app: docker-registry
    spec:
      nodeSelector:
        type: worker
      containers:
      - name: docker-registry
        image: registry:2
        ports:
        - containerPort: 5000
          name: docker-registry
        volumeMounts:
        - mountPath: /var/lib/registry
          name: docker-repo
      volumes:
      - name: docker-repo
        persistentVolumeClaim:
          claimName: docker-repo-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: docker-registry
spec:
  type: NodePort
  ports:
  - port: 5000
    targetPort: docker-registry
    nodePort: 30500
  selector:
    app: docker-registry
```

그런다음 아래 명령으로 배포해주었다.

```bash
kubectl create -f <yaml file>
```

이렇게 registry를 구축했을 경우 이미지를 빌드해서 푸쉬하는건 아래와 같이 하면 된다.

```bash
# docker image build
docker build -t <image name>:<tag> .

# docker local registry tag 부여
docker tag <image name>:<tag> localhost:30500/<image name>:<tag>

# registry로 push
docker push localhost:30500/<image name>:<tag>
```

이렇게 하고 deployment를 작성할때 container image를 `localhost:30500/<image name>:<tag>`로 설정해주면 된다.

> 다만 public release를 위해 최종적으로는 Docker hub를 사용하였다.
{: .prompt-info }

### Kubernetes secret

secret은 쿠버네티스를 사용할때 민감할 수 있는 정보를 따로 secret이라는 객체로 모아서 관리할 수 있게 해주는 기능이다.

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

서비스를 구성하다보면 db 비밀번호나 api key 값들을 사용해야 하는데 이러한 정보를 관리하기 위해 사용해주면 좋다. 시크릿을 생성하는 선언은 아래와 같은 형식으로 해주면 된다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  config.yaml: |
    apiUrl: "https://my.api.com/api/v1"
    username: <user>
    password: <password>
```

> 하지만 쿠버네티스에서 secret이 저장될때는 기본적으로 암호화가 되지 않고 저장한다는 취약점이 있다고 한다. 따라서 암호화등을 고려할 필요 또한 있다.
{: .prompt-danger }

[Kubernetes Secret은 정말 Secret일까?](https://togomi.tistory.com/11)

[Encrypting Confidential Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

프로젝트를 위해 .env 파일을 생성하고 이를 통해 secret을 생성할 수 있도록 구성하였다.

.env 파일 예시

```bash
DOCKER_REGISTRY_URL=<your docker registry url>
DOCKER_USERNAME=<your docker username>
DOCKER_PASSWORD=<your docker password>
DOCKER_EMAIL=<your docker email>

REACT_APP_CLIENT_ID=<your client id>
REACT_APP_CLIENT_SECRET=<your client secret>
REACT_APP_ALOG_API_URL=https://alog.acceler.kr
REACT_APP_YORKIE_API_URL=
REACT_APP_YORKIE_API_KEY=

DB_USERNAME=root
DB_PASSWORD=<your password>

JWT_SECRET=<jwt_secret>
GITHUB_CLIENT_ID=<github_client_id>
GITHUB_CLIENT_SECRET=<github_client_secret>

AWS_ENDPOINT=<aws_endpoint>
REGION=<aws_region>
S3_BUCKET_NAME=<s3_bucket_name>
AWS_ACCESS_KEY_ID=<aws_access_key_id>
AWS_SECRET_ACCESS_KEY=<aws_secret_access_key>

GITHUB_TOKEN=<github_token>
WEBHOOK_SECRET=<webhook_secret>

ARGOCD_SERVER=<argocd_server>
ARGOCD_USERNAME=<argocd_username>
ARGOCD_PASSWORD=<argocd_password>

SPRING_MAIL_HOST=<spring_mail_host>
SPRING_MAIL_PORT=<spring_mail_port>
SPRING_MAIL_USERNAME=<spring_mail_username>
SPRING_MAIL_PASSWORD=<spring_mail_password>
SPRING_MAIL_PROPS_MAIL_SMTP_TIMEOUT=<spring_mail_props_mail_smtp_timeout>
```

secret 생성을 편하게 하기위해 아래와 같이 쉘스크립트 작성

```bash
#!/bin/bash

# Load the variables from the .env file
set -a
source .env
set +a

# Set kubernetes secret from .env file
kubectl create secret generic aform-secret --from-env-file=.env 
kubectl create secret docker-registry regcred --docker-server=${DOCKER_REGISTRY_URL} --docker-username=${DOCKER_USERNAME} --docker-password=${DOCKER_PASSWORD} --docker-email=${DOCKER_EMAIL}
```

### Kubernetes 서비스 설정 파일 작성

kubernetes는 선언적으로 서비스의 구성을 설정해두면 이를 최대한 따르는 방식으로 동작한다.

따라서 앞으로 구축할 서비스를 yaml 파일등으로 구성을 작성하면 이대로 클러스터가 구성되게 된다.

yaml 파일을 작성하기 위해서는 deployment, service, persistent volume 등을 선언해주어야 한다.

## 4. 현재 구조

위 내용을 기반으로 현재까지 구성한 구조는 아래와 같다.

![nodes-final](/nodes-final.png)

![alog-architecture](/alog-architecture.png)

| VM 이름 | Node Name | Type | IP | Spec |
| --- | --- | --- | --- | --- |
| ACCELER_CONTROLPLANE | acceler-controlplane | control-plane | 172.16.211.25 | 2cpu, 16gb ram, 128gb storage |
| ACCELER_node01 | acceler-node-1 | worker, gateway | 172.16.211.26 | 2cpu, 16gb ram, 128gb storage + 64gb ceph storage |
| ACCELER_node02 | acceler-node-2 | worker | 172.16.211.27 | 2cpu, 16gb ram, 128gb storage + 64gb ceph storage |
| ACCELER_node03 | acceler-node-3 | worker | 172.16.211.28 | 2cpu, 16gb ram, 128gb storage + 64gb ceph storage |
| ACCELER_node04 | acceler-node-4 | worker | 172.16.211.29 | 2cpu, 16gb ram, 128gb storage + 64gb ceph storage |

ceph storage를 위한 64gb의 스토리지를 워커노드에 추가로 연결해 주었다.

배포를 위한 설정은 아래 레포에 정리하였다. (위 문서는 A-Log 프로젝트를 기준으로 작성하였다)

A-Form: <https://github.com/KEA-ACCELER/aform-cluster><br>
A-Log: <https://github.com/KEA-ACCELER/alog-cluster>

### 생각해 볼 사항

- master node와 gateway node가 하나의 노드이기 때문에 단일 장애점이 될 위험이 있음
- Secret 관리의 암호화 필요 (cert-manager같은 패키지 사용 고려)
- 서버에 대한 스케일업 필요
