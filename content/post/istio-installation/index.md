---
title: "Instalasi Istio pada Kluster Kubernetes"
date: 2019-07-14
tags: [helm, istio, kubernetes, microservices]
---

By [twahyono](mailto:twahyono@qnp.co.id)

### Apa itu Istio?
Menurut [dokumentasi](https://istio.io/docs/concepts/what-is-istio/), Istio adalah aplikasi yang membantu anda untuk menghubungkan, mengamankan, mengontrol dan memantau _services_ (_microservices_). Istio mengatasi permasalahan kerumitan transisi dari aplikasi monolitik ke aplikasi terdistribusi dengan arsitektur _microservices_.

Ketika aplikasi berkembang kerumitan _microservices_ semakin bertambah dan menyulitkan untuk mengelola dan memahami keterkaitan antara services satu dengan services lainnya. Dari sinilah istilah _services mesh_ muncul. Istilah ini menggambarkan jaringan _microservices_ yang membentuk suatu aplikasi dan interaksi antara services tsb.

Dengan Istio dapat dilakukan hal-hal berikut:

- _Load balancing_ otomatis untuk HTTP, TCP, WebSocket dan gRPC.
- _Traffic management_ dengan _routing rules_, _retries_, _failovers_, _circuit breaker_, _fault injection_.
- Metriks dan monitoring.
- Mengatur keamanan komunikasi antar services.
- _Deployment_ dengan berbagai macam cara: _rollout_, _A/B_, _canary_ dsb.

#### Instalasi Istio dengan Helm
Kali ini kita akan melakukan installasi Istio dengan helm charts. Jika anda belum memasang Helm pada kluster Kubernetes anda, silakan melihat artikel [ini](https://qnpindonesia.github.io/helm-installation). Diasumsikan anda sudah memiliki kluster Kubernetes yang berjalan baik di lokal ataupun di Azure Kubernetes Services (AKS) ataupun Google Kubernetes Engine (GKE).

Tambahkan repo Istio terlebih dahulu:
```
$ helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.1.7/charts/
```

Periksa hasil dari penambahan repo Istio tsb:
```
helm repo list
```

_Output_:
```
NAME            URL
stable          https://kubernetes-charts.storage.googleapis.com
local           http://127.0.0.1:8879/charts
istio.io        https://storage.googleapis.com/istio-release/releases/1.1.7/charts/
```

Kemudian install _Custom Resource Definition_ Istio (CRD) dengan chart istio-init:
```
$ helm install --name istio-init --namespace istio-system istio.io/istio-init
```

Anda akan mendapatkan _output_ seperti berikut:
```
NAME:   istio-init
LAST DEPLOYED: Sun Jul 14 23:22:36 2019
NAMESPACE: istio-system
STATUS: DEPLOYED
```

Sekarang lakukan instalasi Istio dengan helm chart. Sebaiknya turut diinstall juga grafana, aplikasi untuk menampilkan dashboard serta monitoring metriks serta kiali, aplikasi untuk melihat visualisasi keterkaitan antar microservices. Aktifkan pilihan konfigurasi dengan ```--set grafana.enabled=true``` atau ```--set kiali.enabled=true```.

```
$ helm install --name istio --namespace istio-system --set grafana.enabled=true istio.io/istio
```

Periksa _pods_ yang muncul dari hasil instalasi Istio:
```
kubectl get pods -n istio-system
```

Jika berhasil maka terlihat seperti berikut:
```
NAME                                      READY     STATUS      RESTARTS   AGE                                          grafana-85689d5548-6zbtc                  1/1       Running     0          15h
grafana-85689d5548-ctzvg                  1/1       Running     0          15h
istio-citadel-5fdf5ccf85-mkt2l            1/1       Running     0          15h
istio-egressgateway-6f887d4d46-vkvjh      1/1       Running     0          15h
istio-galley-88c6c64b7-49hmz              1/1       Running     0          15h
istio-ingressgateway-7646456c5c-hdvrl     1/1       Running     0          1d
istio-init-crd-10-sh7dn                   0/1       Completed   0          6m
istio-init-crd-11-64nlt                   0/1       Completed   0          6m
istio-pilot-65bd87cb4c-8s4zp              2/2       Running     0          1d
istio-policy-55cdb54666-hvshp             2/2       Running     0          15h
istio-sidecar-injector-596f6bf5d5-qbl6k   1/1       Running     0          1d
istio-telemetry-7dbb44954f-k2vgc          2/2       Running     0          15h
istio-tracing-d444f578-mc4b4              1/1       Running     1          1d
kiali-5bdc5b4d69-89lqn                    1/1       Running     0          15h
kiali-5bdc5b4d69-cc624                    1/1       Running     0          1d
prometheus-6c56b9bf49-46z97               1/1       Running     0          15h
```

Istio menggunakan mekanisme sidecar untuk menginstall Envoy proxy pada tiap pod yang dijalankan di Kubernetes. Untuk melakukan otomatisasi hal ini perlu ditambahkan label istio-injection=true untuk namespace yang dituju:

```
kubectl label namespace default istio-injection=enabled
```

Kemudian periksa hasil dari pelabelan tsb
```
kubectl get namespaces -L istio-injection
```

_Output_:
```
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    31d       enabled
istio-system   Active    31d
kube-public    Active    31d
kube-system    Active    31d
```

Instalasi Istio sudah berhasil. Di artikel selanjutnya kita akan membahas bagaimana melakukan konfigurasi Istio untuk berbagai keperluan. 