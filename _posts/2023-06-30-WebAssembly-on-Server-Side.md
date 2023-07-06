---
title: WebAssembly on Server Side
author:
  name: chromato99
  link: https://chromato99.com/about/
date: 2023-06-30 21:00:00 +0900
img_path: /assets/img/posts/2023-06-30-WebAssembly-on-Server-Side/
categories: [Cloud Native, WASM]
tags: [linux, wasm, cloudnative, serverless]
---


## WebAssembly란?

[WebAssembly](https://webassembly.org/)

[WebAssembly - MDN](https://developer.mozilla.org/en-US/docs/WebAssembly)

WebAssembly(WASM)는 최신 웹 브라우저에서 실행할 수 있는 새로운 유형의 코드이다. 네이티브에 가까운 성능으로 동작하며 컴팩트한 바이너리 포맷을 제공하는 저수준 어셈블리 언어로, C/C++, Rust 등과 같은 언어의 컴파일 타겟으로써 그런 언어로 작성된 프로그램을 웹에서 사용할 수 있게 해준다.

이는 자바스크립트만 실행 가능한 웹브라우저에서 자바스크립트의 한계를 극복하기 위해 등장하였다. 

기존에는 웹 브라우저에서 실행되는 스크립트 언어인 JavaScript를 사용하여 웹 애플리케이션을 개발했는데, JavaScript는 동적인 웹 콘텐츠를 만드는 데 적합하지만, CPU 집약적인 작업이나 고성능의 알고리즘이 필요한 경우에는 제한된 성능을 보였다.

그러나 WebAssembly가 등장하며 웹 개발자들은 WebAssembly를 사용하여 기존의 CPU 집약적인 작업을 훨씬 빠르게 실행하고, 3D 그래픽, 오디오 처리, 비디오 인코딩 등의 작업을 웹에서 처리할 수 있게 되었다. 

## WebAssembly on Server Side

### 웹 브라우저 밖에서의 WebAssembly

[[번역] 웹어셈블리에 주목하라](https://medium.com/@yujso66/번역-웹어셈블리에-주목하라-280ff4e9ce01)

[웹어셈블리(Wasm)가 클라우드 컴퓨팅의 미래인 이유](https://www.itworld.co.kr/news/263104)

[WebAssembly에 주목해야 할 이유 - GeekNews](https://news.hada.io/topic?id=5914)

![wasmcloud-k8s-edge](https://cosmonic.com/assets/images/wasmcloud-k8s-edge-c3b96490c9f09243b24de51765b70bfa.png)

WebAssembly를 웹브라우저에서만 사용하는 것에서 더 나아가 서버사이드와 Cloud Native 환경에서 WebAssembly를 사용하는 것이 주목받고 있다.

우선 웹 어셈블리의 장점은 다음과 같다. 

웹어셈블리는 자바스크립트 엔진에서 동작하므로 자바스크립트 엔진이 동작하는 어떤 웹 브라우저나 기기에서 동작할 수 있다. 

또한 사전에 바이트코드로 컴파일 되었기 때문에 네이티브로 컴파일 되는 환경보다는 느리지만 자바스크립트보다는 훨씬 빠르게 동작할 수 있다. 그리고 다양한 언어로 작성된 코드를 웹에서 실행할 수 있으며, 기존의 코MDN드를 웹에 쉽게 재사용할 수 있다. 

마지막으로 activex나 어도비 플래쉬 등의 보안 문제를 겪어오면서 웹브라우저의 보안은 매우 발전하였는데, 이과정에서 자바스크립트 엔진은 샌드박스 환경을 제공해 코드의 실행을 격리하고 보안을 유지한다. 웹 어셈블리는 이 자바스크립트 엔진에서 구동되는 만큼 이러한 보안의 장점을 그대로 가져갈 수 있다.

그렇다면 이러한 장점이 많은 웹 어셈블리를 클라이언트 측인 웹브라우저에서만 사용하는 것이 아니라 분리하는 것을 생각해 볼 수 있다. 우수한 이식성, 빠른 성능, 여러 언어를 사용할 수 있다는 장점, 오랜시간동안 성숙된 보안은 조금만 생각해보면 서버를 구동하기 위한 아주 좋은 조건이라는 것을 알 수 있다. 여기서 웹 어셈블리를 서바사이드에서 사용하는것을 생각해볼 수 있다.

![wasm-non-web-embedding](https://megaease.com/imgs/blogs/wasm.01.png)

이러한 서버사이드에서의 활용을 위해 WebAssembly는 non-web embedding을  WASI (WebAssembly System Interface)를 통해 지원하고 있다.

### WASI

WASI는 WASM을 비웹 환경에서 실행하기 위한 표준 인터페이스이다. WASI는 파일 시스템, 네트워킹, 암호화 등과 같은 시스템 리소스에 대한 접근을 WASM 모듈에 제공한다. WASI는 호스트 환경의 기능을 WASM 모듈에 노출시킴으로써 WASM이 비웹 환경에서도 시스템 리소스와 상호 작용할 수 있게 한다.

WASI를 사용하여 WASM을 비웹 환경에 임베딩하는 경우, WASM 모듈은 WASI 인터페이스를 통해 호스트 환경의 리소스에 접근하고 함수 호출을 수행할 수 있다. 이를 통해 WASM 모듈은 파일 시스템 조작, 네트워킹, 시스템 호출 등과 같은 작업을 수행할 수 있다.

### Low-level VM and Containerization

정리하면 WASM과 WASI는 웹 브라우저 밖에서 샌드박싱 환경을 제공하는 일종의 Low-level VM 환경이라 볼 수 있다. 그리고 이러한 WASM이 컨테이너화 된다면 도커와 비슷한 또는 도커를 대체할 수 있는 서버 사이드 컴퓨팅의 새로운 미래가 될 수 있을 것으로 주목받고 있다.

실제로 최근 Docker에서는 docker + wasm 기능을 베타로 공개하였다. 

[Wasm workloads (Beta)](https://docs.docker.com/desktop/wasm/)

[Introducing the Docker+Wasm Technical Preview - Docker](https://www.docker.com/blog/docker-wasm-technical-preview/)

[Build, Share, and Run WebAssembly Apps Using Docker](https://www.docker.com/blog/build-share-run-webassembly-apps-docker/)

이는 docker 플랫폼에서 linux container 기술을 사용해 컨테이너 이미지를 생성하는 것이 아닌 WebAssembly를 기반으로 컨테이너를 생성하고 운용할 수 있는 기능이다. 

![docker-containerd-wasm-diagram](https://www.docker.com/wp-content/uploads/2022/10/docker-containerd-wasm-diagram.png.webp)

Docker는 여기서 컨테이너를 구동하는 하위 런타임인 runc를 wasmedge로 변경하는 방식으로 containerd의 규격하에 wasmedge가 구동될 수 있도록 했다고 한다. 이를 통해 기존의 컨테이너가 웹어셈블리 모듈과 함께 containerd라는 표준 환경 아래에서 구동될 수 있다.

Docker + Wasm을 사용하려면 containerd 이미지 저장소 기능을 활성화하고 Docker Desktop 설정에서 Wasm 런타임을 설치해야 한다. 그런 다음 --runtime 및 --platform 플래그를 사용하여 docker run 또는 docker compose로 Wasm 컨테이너를 실행할 때 Wasm 런타임 및 아키텍처를 지정할 수 있다. 또한 Docker Compose를 사용하여 Wasm 컨테이너를 다른 Linux 컨테이너와 연결하여 Wasm으로 다중 서비스 애플리케이션을 실행할 수 있다.

현재는 테크니컬 프리뷰로 향후 버전에서는 많은 부분이 변경될 수 있다. 자세한것은 위 링크를 참고하자.

## 기존 컨테이너 기술과의 비교

[Will Cloud-Native WebAssembly Replace Docker?](https://kubesphere.io/blogs/will-cloud-native-webassembly-replace-docker_/)

[Why Containers and WebAssembly Work Well Together - Docker](https://www.docker.com/blog/why-containers-and-webassembly-work-well-together/)

![wasm vs docker](https://miro.medium.com/v2/resize:fit:1400/1*70lMc8l4hRh8mllatYKEhQ.png)

기존의 컨테이너 기술은 운영체제 커널은 공유하므로써 VM보다 효율적이었지만 그위에 올라가는 수많은 Binary와 Library는 중복해서 설치해야 하는 문제가 있다.

하지만 WASM은 사실상 웹브라우저에서 자바스크립트 엔진만 분리한거기 때문에 가볍다. (웹브라우저 실행하는데 얼마 안걸리는 것에서 알 수 있음) 그리고 웹브라우저는 보안을 위한 샌드박스 환경을 제공하도록 되어있기 떄문에 일종의 격리환경이라 볼 수 있다. 

따라서 기존의 Container 보다 훨씬 빠르면서도 격리된 환경을 제공할 수 있다. 

![solomon-hykes](/solomon-hykes.png)

실제로 Docker 설립자 Solomon Hykes는 웹어셈블리의 중요성을 강조하기도 하였다.

### WASM의 장점

WebAssembly를 서버에서 사용할때 장점을 정리하면 아래와 같다.

- 포터블: 표준화된 바이트코드로 브라우저/서버 어디서든 실행
- 유니버설: C,Rust,Go,Python,Ruby등 다양한 언어들이 Wasm으로 컴파일
- 네이티브 수준의 성능: 평균적으로 네이티브보다 1.45~1.55배 느리지만 JavaScript보다는 항상 빠름
- 빠른 시작 시간: 도커 컨테이너보다 10~100x 빠르고, 브라우저에서도 Javascript 파싱/인터프리팅 보다 빠름
- 안전: 웹을 염두에 두고 개발되어 메모리 샌드박싱 및 기능 제한 등의 보안기능 제공

여러 장점들로 인해 WASM은 많은 분야에서 주목받고 있는데, 특히 Cloud Native 환경을 위한 Kubernetes와 Serverless Computing이다. 

실제로 이미 Microsoft Azure의 Deis Labs는 기존 Kubernetes 클러스터에서 Wasm 작업을 실행하는 방법인 [Krustlet](https://krustlet.dev/)이 개발중이다.

또한 빠른 콜드 스타트 덕분에 Serverless 컴퓨팅에서 기존의 container 기술을 대체할 기술로 조목받고 있다.

### WASM 현재 단계에서의 한계

하지만 아직 한계점 또한 존재하는데 정리하면 아래와 같다.

- 웹어셈블리 표준화는 아직 없다. 예를 들어 웹어셈블리 시스템 인터페이스(WebAssembly System Interface)에는 공식적으로 표준화되지 않은 수많은 확장이 있지만 다양한 런타임에서 이러한 확장을 선택하여 구현하고 있다. 보편적인 이식성에 대한 약속이 아직은 부족하다.
- 언어 간 상호 작용이 아직 형편없다. 사람들이 실제로 여러 언어에서 Wasm을 사용하기 시작하기 전에 중요한 언어에 대한 웹어셈블리 컴포넌트와 우수한 코드 생성기가 필요하다.
- 개발자 경험이 많이 부족하다. 특히 디버깅과 패키지 매니저, 빌드 시스템 및 IDE와의 통합과 관련된 도구 개선이 필요하다.

하지만 이러한 제약에도 불구하고 웹 쪽에서는 TensorFlow.js, Unity, Figma, AutoCAD 및 Squoosh 등에서 이미 사용되고 있고, 또 빠르게 발전하고 있는 만큼 서버 쪽에서의 한계도 빠르게 개선될것이라 기대된다.

## Docker + WASM Demo

마지막으로 주목 받고 있는 WASM을 Docker와 함께 사용하는 Demo를 알아보았다.

[microservice-rust-mysql](https://github.com/second-state/microservice-rust-mysql)

위 링크는 mysql (실제 데모에서는 mardiadb) 도커 컨테이너와 함께 rust로 작성된 WASM 서버가 함께 동작하는 것을 테스트 해볼 수 있는 데모이다. 이는 WASM 런타임과 docker container가 docker engine위에서 함께 동작하는 것을 테스트 해보는 것으로 앞으로 웹 어셈블리가 기존의 컨테이너 환경과 함께 유기적으로 동작할 수 있다는 것을 확인할 수 있다. 

자세한 실행법은 링크에서 확인 할 수 있다. 아래는 실제 데모가 구동되는 모습이다.

![demo-1](/demo-1.png)

Docker Desktop에서 Beta 활성화

![demo-2](/demo-2.png)

`docker compose up` 명령을 사용했을때 서버가 동작하는 모습

![demo-3](/demo-3.png)

데모 페이지

![demo-4](/demo-4.png)

데이터 삽입 테스트

![demo-5](/demo-5.png)

데이터 확인
