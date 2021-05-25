Jenkins CI/CD 도입을 위한 Information 정리
=======================================
# 1. CI/CD
## 1.1 Continuous Integration(지속적 통합)
 팀단위의 개발을 하는 단체에서는 반드시 지녀야 하는 것
 글자 그대로, 여러 개발자의 소스를 한날 한시에 도원결의 마냥 통합하는 것이 아닌 지속적으로 통합하여 그때 그때 발생하는 문제점들을 agile 하게 해결해 나가는 행위
## 1.2 Continuous Distribution(지속적 배포)
 개발, 통합, 테스트, 배포, 릴리즈를 자동화하여 지속적으로 최신화된 제품을 생산하는 행위.
 개발부터 릴리즈까지 자동화를 동반해야 유의미하게 흘러갈 수 있다.
***
# 2. Jenkins
 CI 자동화 툴들 중 하나로, 04년 썬 마이크로시스템즈에서 개발이 시작되어 05년에 java.net 에 처음 출시 되었다.
 Jenkins 와 같은 CI 툴들이 등장하기 이전에는, 개발자들이 commit 이 끝난 심야시간대에 build 를 진행하는 일명 'nightly-build' 가 이루어졌었다고 한다.
 CI 자동화 툴을 이용함으로서, 개발자들은 기존에 개발 -> 테스트 -> 빌드 하는 등의 번거로운 과정에서 소스를 commit 후 퇴근하면, 다음 날 버그 리포트를 기반으로 작업을 하는 간소화된 개발 루틴을 가질 수 있다.
 이 중 Jenkins 는 Github, Gitlab 과 같은 VCS 들과의 연동을 간편하게 할 수 있어, 사내망에 Jenkins 를 이용하여 CI/CD 자동화 환경을 구성해 보았다.
***
# 3. 도입 예시
## 3.1 기저 개발 환경
 도입에 앞서, 개발 환경을 간략히 보면 다음과 같다.
 1. Gitlab (Community Edition, Version : 9.5.0, IP 주소 : 10.1.x.x)
 2. 만들어진 이미지들을 배포할 배포서버 (IP 주소 : 10.77.x.x)
 3. Local PC (IP 주소 : 10.77.x.x)
 4. Jenkins 가 설치될 서버 (Ubuntu 18.04 LTS, Vagrant 2.2.14)
 
 아이템 별로 분산 빌드를 구축하기 위해, Vagrant 로 Jenkins 서버 구성을 하여 Master/Agent(Slave) 구조로 구축하였다.
## 3.2 Vagrant 구성
 Gitlab 서버(10.1.x.x) 와 배포 서버 및 Local PC(10.77.x.x) 대역과 통신이 되어야 하므로 Vagrant 로 구성 시 IP 를 2개 할당해준다.
 "public_network" 2개를 명시해주면 2개의 NI 가 생긴다.
 
 ```rust
 config.vm.define "jenkins" do |node|
      node.vm.hostname = "jenkins"
      node.vm.network "public_network", ip: "10.1.10.141", bridge: "enx705dccfa0b19"
      node.vm.network "public_network", ip: "10.77.10.5", bridge: "enx705dccfa0f13"
      node.vm.synced_folder "src/", "/home/vagrant/sources", type: "rsync"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "jenkins"
        vb.gui = false
        vb.cpus = 2
        vb.memory = "2048"
      end
  end
  ```
  
 상기의 노드는 Master/Agent 중 Master 노드의 역할을 한다. Agent 노드도 마찬가지로 10.1.x.x 대역과 10.77.x.x 대역 2개의 IP를 모두 할당해준다.
 
***
# References
[Jenkins 기본 Port 변경](https://jojoldu.tistory.com/354)

[CI/CD(지속적 통합/지속적 제공): 개념, 방법, 장점, 구현 과정](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)

[Gitlab Private Repository Jenkins 연동](https://softwaree.tistory.com/66)

> gitlab 연동 시, 'Gitlab API Token' 은 제대로 Add 가 되지 않으므로, 'Username with password' 로 Credential 을 만들어야 한다.

[Gitlab Webhook 404 Not Found Error](https://github.com/jenkinsci/gitlab-plugin/issues/608)

[License 문제 해결](https://beomseok95.tistory.com/185)

[Agent(Slave) Node 추가하여 분산빌드(Distributed Build) 환경 구성하기](https://nirsa.tistory.com/302)
