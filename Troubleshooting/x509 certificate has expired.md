# Kubernetes 인증서 교체하기

## 오류 내용
Unable to connect to the server: x509: certificate has expired or is not yet valid: current time 2024-04-03T08:56:42+09:00 is after 2024-03-30T05:13:23Z

### 인증서 교체 절차


1. 인증서 만료 기한 확인
```bash
$ kubeadm certs check-expiration
```

2. 기존 인증서 백업
```bash
$ cp -pr /etc/kubernetes/ /etc/kubernetes_backup
```

3. 인증서 갱신
```bash
$ kubeadm certs renew all
```

4. config 파일 교체
```bash
$ cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

5. 서비스 재가동(인증서 적용)
```bash
$ kill -s SIGHUP $(pidof kube-apiserver)
$ kill -s SIGHUP $(pidof kube-controller-manager)
$ kill -s SIGHUP $(pidof kube-scheduler)
$ systemctl restart kubelet
$ systemctl daemon-reload
```

6. 인증서 갱신 확인
```bash
$ kubeadm certs check-expiration
```