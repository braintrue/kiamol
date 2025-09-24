# Chapter 2 Lab - Kubernetes Deployment 실습 기록

이 실습은 `whoami` 애플리케이션을 Kubernetes 클러스터에 배포하고,  
Pod에 접근하여 동작을 확인하는 과정을 담고 있습니다.  

---

## 1. Deployment 생성
```bash
kubectl apply -f solution/deployment.yaml

#출력
deployment.apps/whoami created

kubectl port-forward deploy/whoami 8080:80
curl http://localhost:8080

> "I'm whoami-687976f48b-tkxp9 running on Linux 4.19.76-linuxkit #1 SMP Thu Oct 17 19:31:58 UTC 2019"

kubectl get pods -o custom-columns=NAME:metadata.name
> whoami-68bf776fd-s6sr9

kubectl exec deploy/whoami -- sh -c 'hostname'
>whoami-687976f48b-tkxp9

📌 배운 점

Deployment를 생성하면 Kubernetes가 자동으로 Pod을 관리한다.

kubectl port-forward를 통해 로컬 포트와 Pod 포트를 연결하여 접근할 수 있다.

kubectl exec로 Pod 내부 명령어를 실행할 수 있다.

Pod 이름은 재시작될 때마다 달라지지만, Deployment로 관리하면 항상 원하는 상태가 유지된다.

> whoami-68bf776fd-s6sr9

kubectl exec deploy/whoami -- sh -c 'hostname'

> whoami-687976f48b-tkxp9

braintrue/kiamol/
└── ch02/
    └── lab/
        ├── README.md        # 실습 기록
        └── solution/
            └── deployment.yaml
