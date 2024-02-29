# **Client IP(x-real-ip) Issue**

### 상태: Client <-> MetalLB <-> Ingress-Nginx-Controller <-> API Server
`Client`에서 보낸 HTTP(S) 메시지를 `API Server`에서 봤을 때 아래와 같이 x-real-ip 값이 `Client`의 IP가 아닌 Load Balancer의 IP로 되어 있는 것을 확인할 수 있다.
```json
Headers: {
  "host": "***.*******.com",
  "x-request-id": "9976e980124ecefcf8a9c5b8f79320f4",
  "x-real-ip": "10.244.0.1",                                    <-------- this
  "x-forwarded-for": "10.244.0.1",
  "x-forwarded-host": "***.*******.com",
  "x-forwarded-port": "443",
  "x-forwarded-proto": "https",
  "x-forwarded-scheme": "https",
  "x-scheme": "https",
  "content-length": "473",
  "sec-ch-ua": ""Chromium";v="122", "Not(A:Brand";v="24", "Google Chrome";v="122"",
  "sec-ch-ua-platform": ""Windows"",
  "sec-ch-ua-mobile": "?0",
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
  "content-type": "application/json",
  "accept": "*/*",
  "origin": "https://***.*******.com",
  "sec-fetch-site": "same-origin",
  "sec-fetch-mode": "cors",
  "sec-fetch-dest": "empty",
  "referer": "https://***.*******.com/****/******",
  "accept-encoding": "gzip, deflate, br, zstd",
  "accept-language": "ko,en;q=0.9,ru;q=0.8,en-US;q=0.7,ko-KR;q=0.6",
  "cookie": "__stripe_mid=b48812c1-33c1-4db5-bac4-caf979cbdf00fca66d; refresh_token=eyJh...; access_token=..."
}
```

해당 문제를 해결하기 위해서는 `Ingress-Nginx-Controller`의 `ConfigMap`과 `Service`를 수정해야 한다.
먼저 `ConfigMap`는 `enable-real-ip`과 `proxy-real-ip-cidr` 설정을 아래와 같이 수정한다.
`proxy-real-ip-cidr`는 Load Balancer가 속한 서브넷의 CIDR Block 입력한다.
```yaml
apiVersion: v1
data:
  allow-snippet-annotations: "true"
  proxy-body-size: "50m"
  enable-real-ip: "true"                                 <--------- this
  proxy-real-ip-cidr: "10.244.0.0/24"                    <--------- this
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.7.0
  name: ingress-nginx-controller
  namespace: ingress-nginx
```

다음 `Service`는 `externalTrafficPolicy: Local` 설정을 아래와 같이 추가해야 한다.
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.7.0
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: LoadBalancer
  externalTrafficPolicy: Local                       <--------- this
```


수정 후 재시작(적용)하면 아래와 같이 `Client`의 IP가 `x-real-ip`로 제대로 전달되는 것을 확인할 수 있다.
```json
Headers: {
  "host": "***.********.com",
  "x-request-id": "938764f1a8cecfc0272e5833e6ec8efe",
  "x-real-ip": "61.43.202.254",
  "x-forwarded-for": "61.43.202.254",
  "x-forwarded-host": "***.********.com",
  "x-forwarded-port": "443",
  "x-forwarded-proto": "https",
  "x-forwarded-scheme": "https",
  "x-scheme": "https",
  "content-length": "473",
  "sec-ch-ua": ""Chromium";v="122", "Not(A:Brand";v="24", "Google Chrome";v="122"",
  "sec-ch-ua-platform": ""Windows"",
  "sec-ch-ua-mobile": "?0",
  "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
  "content-type": "application/json",
  "accept": "*/*",
  "origin": "https://***.********.com",
  "sec-fetch-site": "same-origin",
  "sec-fetch-mode": "cors",
  "sec-fetch-dest": "empty",
  "referer": "https://***.********.com/****/****",
  "accept-encoding": "gzip, deflate, br, zstd",
  "accept-language": "ko,en;q=0.9,ru;q=0.8,en-US;q=0.7,ko-KR;q=0.6",
  "cookie": "__stripe_mid=b48812c1-33c1-4db5-bac4-caf979cbdf00fca66d; refresh_token=eyJh...; access_token=eyJ..."
}
```