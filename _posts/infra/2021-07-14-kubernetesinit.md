---
layout: post
title: kubernetes 기본 설치 및 dashboard 배포
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Infra, k8s]
categories : [Infra]
---

### kubernetes 기본 설치 및 dashboard 배포 방법

1. Docker 설치
    ```
	$ sudo apt-get update
	$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
	$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	$ sudo add-apt-repository \
	"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	$(lsb_release -cs) \
	stable"
	$ sudo apt-get update
	$ sudo apt-get install docker-ce docker-ce-cli containerd.io

	1-1. Docker 설정

	$ docker info | grep -i cgroup 
	$ cat <<EOF> /etc/docker/daemon.json < { "exec-opts" : [ "native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": { "max-size" : "100m" },
	"storage-driver": "overlay2",
	"storage-opts": [ "overlay2.override_kernel_check=true",
	"insecure-registries": ["<IP>:4567"]
	}
	EOF
	$ systemctl daemon-reload
	$ systemctl restart docker
	```

2. Swap Off

	kubernetes는 swap을 사용하지않는 것을 권장한다.

	```
	$ swapoff -a
	$ vim /etc/fstab
	ubuntu의 경우 swap.img 해당라인 주석 처리
	$ reboot
	```

3. Kubenetes 설치
	```
	$ sudo apt install apt-transport-https
	$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
	$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
	$ sudo apt update
	$ sudo apt install kubelet=1.19.5-00 kubeadm=1.19.5-00 kubectl=1.19.5-00 kubernetes-cni=0.8.7--00
	```

	1.20이후로 docker runtime대신 containerd나 다른 CRI를 지원하게 된다고 한다. 1.22에서 더이상 지원되지않는다고 함
	현재 버전에서 deprecate 되었는지 모르겠으나 1.19로 일단 편하게..
	[참조](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

4. Kubenetes Master Cluster 구성

	4-1. kubeadm 을 이용하여 master 노드 구성
		```
		$ kubeadm init --pod-network-cidr=<IP>/16 --apiserver-advertise-address=<host ip>

			• --pod-network-cidr 의 IP는 host ip와 다르게 해줘야 한다.
			• 현재 사용중인 CNI Calico의 설정은 172.126.0.0/16
		```

	4-2. KUBECONFIG 권한 설정
		```
		$ export KUBECONFIG=/etc/kubernetes/admin.conf // 임시로 설정
		$ cp /etc/kubernetes/admin.conf ~/.kube/config // 로그인된 사용자가 항상 kubectl를 사용할 수 있도록 kubeconfig 파일을 복사

		4-3 node 확인

		kubectl get node


		4-3. CNI 설치 (calico 이용)

		$ kubectl apply -f calico/calico.yaml


		4-4. Master 노드 pod 생성 방지 해제

		$ kubectl taint nodes --all node-role.kubernetes.io/master-

		4-5. Worker node의 Master node join

			• Join command 조회
		$ kubeadm token create --print-join-command

		$ kubeadm join <IP>:6443 --token us5yob.zif032vatynjeci3     --discovery-token-ca-cert-hash sha256:480bd892953aa0fc8b538fb5e5e90263a67ffe4060b2be3e26fb48b6f8e71e63
		```

5. Dashboard 설치

	```
	$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml 을 하거나 
 
	$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml를 다운받아

	$ kubectl create -f recommended.yaml 

	$ kubectl delete -f filename이나 {resource name}으로 삭제 가능
	```

	로 설치 가능

	접속은 8443 port로 가능하다.

	해당 포트는 service의 NodePort 설정으로 변경할 수 있다.

6. kubernetes 상태 확인
	```
	모든 pod 상태 확인  
	$ kubectl get pod -A -o wide

	-o wide는 옵션으로 더 자세하게 보여준다.

	모든 service 확인  
	$ kubectl get service -A

	이처럼 kubectl get {resource name} -n {namespace}

	으로 상태 확인을 할 수 있다. 

	edit을 통해 수정도 가능

	kubectl describe {resource name} -n {namespace} 로 자세하게 해당 리소스의 상태를 확인가능하다.
	```

---

이외에 helm 이라는  package managing tool을 이용해 다양한 서비스를 편하게 배포할 수 있다.