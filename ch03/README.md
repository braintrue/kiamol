#실습 환경 설정 완료 #디렉터리로 이동하기
cd ch03

#두 개의 디플로이먼트 생성: 각각 파드 하나를 실행합니다.
kubectl apply -f sleep/sleep1.yaml -f sleep/sleep2.yaml

> deployment.apps/sleep-1 created
> deployment.apps/sleep-2 created

#파드가 완전히 시작할 때까지 기다립니다.
kubectl wait --for=condition=Ready pod -l app=sleep-2

#두 번째 파드의 IP 주소를 확인합니다.
kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'

#같은 주소를 사용하여 두 번째 파드에서 첫 번째 파드로 ping을 보냅니다.
kubectl exec deploy/sleep-1 -- ping -c 2 $(kubectl get pod -l app=sleep-2 --output jsonpath='{items[0].status.podIP}')

> PING 10.42.0.40 (10.42.0.40): 56 data bytes
> 64 bytes from 10.42.0.40: seq=0 ttl=64 time=0.515 ms
> 64 bytes from 10.42.0.40: seq=1 ttl=64 time=0.202 ms

--- 10.42.0.40 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.202/0.358/0.515 ms

#파드의 현재 주소를 확인합니다.
kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'

> 10.42.0.40%

#현재 파드를 삭제해 디플로이먼트가 새 파드를 생성하도록 합니다.
kubectl delete pods -l app=sleep-2

> pod "sleep-2-d4446db4d-wfbhz" deleted

#같은 명령어! 새로 대체된 파드의 IP 주소를 확인합니다.
kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'

> 10.42.0.41%

#서비스를 배포합니다! yaml 파일 + kubectl의 apply 명령을 사용합니다.
kubectl apply -f sleep/sleep2-service.yaml

> service/sleep-2 created

#서비스의 상세 정보를 출력합니다.
kubectl get svc sleep-2

> NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
> sleep-2 ClusterIP 10.43.58.39 <none> 80/TCP 18s

#파드와 통신이 잘 되는지 확인합니다. ping 명령은 ICMP 프로토콜을 사용하지만, 서비스 리소스는 표준 TCP, UDP 프로토콜을 지원하기 때문입니다.
kubectl exec deploy/sleep-1 -- ping -c 1 sleep-2

> PING sleep-2 (10.43.58.39): 56 data bytes

--- sleep-2 ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
command terminated with exit code 1

#웹사이트, API를 담당할 두 개의 디플로이먼트를 실행합니다.
kubectl apply -f numbers/api.yaml -f numbers/web.yaml

> deployment.apps/numbers-api created
> deployment.apps/numbers-web created

#파드의 준비가 끝날 때까지 기다립니다.
kubectl wait --for=condition=Ready pod -l app=numbers-web

> pod/numbers-web-6956b74d78-csq82 condition met

#웹 어플리케이션에 포트 포워딩을 적용합니다.
kubectl port-forward deploy/numbers-web 8090:80

> Forwarding from 127.0.0.1:8090 -> 80
> Forwarding from [::1]:8090 -> 80
> Handling connection for 8090 #웹 브라우저에서 https://localhost:8090에 접근하여 화면상의 Go 버튼을 클릭하면 오류가 발생합니다. #웹 애플리케이션 파드, API 파드를 배포했지만 서비스는 생성하지 않아 두 컴포넌트가 서로 통신하지 못하기 때문입니다.
> #KIAMOL Random Number Generator
> #RNG service unavailable!
> #(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng) #오류 메시지: API 접근 불가 #해당 API의 도메인 네임은 numbers-api, 해당 도메인 네임은 쿠버네티스 내부의 DNS 서버에 등록되어 있지 않습니다.

#서비스를 배포합니다. api.yaml -> api-service.yaml
kubectl apply -f numbers/api-service.yaml

> service/numbers-api created

#서비스의 상세 정보를 출력합니다.
kubectl get svc numbers-api

> NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
> numbers-api ClusterIP 10.43.146.245 <none> 80/TCP 95s

#웹 애플리케이션에 접근할 수 있도록 포트 포워딩을 적용합니다.
kubectl port-forward deploy/numbers-web 8090:80

> Forwarding from 127.0.0.1:8090 -> 80
> Forwarding from [::1]:8090 -> 80
> Handling connection for 8090

#https://localhost:8090

> KIAMOL Random Number Generator
> #API에서 생성한 무작위 숫자 7을 읽어 와 화면에 출력합니다.
> Here it is: 7
> (Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)

#API 파드의 이름과 IP 주소를 확인합니다.
kubectl get pod -l app=numbers-api -o custom-columns=NAME:metadata.name,POD_IP:status.podIP

> NAME POD_IP
> numbers-api-69d58f875b-q9gcv 10.42.0.42

#API 파드를 수동으로 삭제합니다.
kubectl delete pod -l app=numbers-api

> pod "numbers-api-69d58f875b-q9gcv" deleted

#새로 생성된 대체 파드의 이름과 IP 주소를 확인합니다.
kubectl get pod -l app=numbers-api -o custom-columns=NAME:metadata.name,POD_IP:status.podIP

> NAME POD_IP
> numbers-api-69d58f875b-tflv8 10.42.0.44

#웹 애플리케이션에 접근할 수 있도록 포트 포워딩을 적용합니다.
kubectl port-forward deploy/numbers-web 8090:80

> Forwarding from 127.0.0.1:8090 -> 80
> Forwarding from [::1]:8090 -> 80
> Handling connection for 8090

#https://localhost:8090

> KIAMOL Random Number Generator
> #API에서 생성한 무작위 숫자 7을 읽어 와 화면에 출력합니다.
> Here it is: 7
> (Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng) #계속해서 API 파드와 통신할 수 있다는 점을 확인할 수 있습니다.
