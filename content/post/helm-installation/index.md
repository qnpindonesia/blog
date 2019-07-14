---
title: "Bagaimana Memasang Helm pada Kluster Kubernetes"
date: 2019-07-14
tags: [helm, kubernetes]
---

By [twahyono](mailto:twahyono@qnp.co.id)

### Apa itu Helm?
Helm adalah _package manager_ untuk Kubernetes, sebagaimana npm adalah _package manager_ untuk JavaScript di  dalam lingkungan Node.js atau seperti _package manager_ [Chocolatey](https://chocolatey.org) untuk Windows. Helm memudahkan tim DevOps melakukan _deployment_ aplikasi ke dalam kluster Kubernetes.

Helm sendiri terdiri dari dua bagian: _Helm client_ (helm) dan _Helm server_ (Tiller). Ikuti langkah-langkah di bawah ini untuk memasang Helm pada kluster Kubernetes anda.

### Persiapan Awal
1. Kluster Kubernetes yang telah berjalan.
1. _Tools_ kubectl di mesin lokal.

### Langkah 1
Lakukan hal berikut di _shell_ Linux. Masuk ke direktori /tmp kemudian unduh _script_ dari repo Helm di GitHub:

```
$ cd /tmp
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > install-helm.sh
```

Agar _script_ tsb dapat dijalankan dengan gunakan perintah ```chmod```:
```
$ chmod u+x install-helm.sh
```

Lanjutkan dengan menjalankan _script_ berikut:
```
$ install ./install-helm.sh
```

_Output_:
```
helm installed into /usr/local/bin/helm
Run 'helm init' to configure helm.
```

Selanjutnya anda perlu memasang komponen Tiller pada kluster Kubernetes.

### Langkah 2 - Memasang Tiller
#### Apa itu Tiller?
Tiller merupakan komponen yang berjalan di dalam kluster Kubernetes dan bertanggung jawab untuk melakukan interaksi dengan Helm client dan melakukan proses antarmuka dengan Kubernetes API Server. Singkatnya Helm client mengelola _helm charts_ dan Tiller akan mengelola _release_. Pada Helm versi 3 yang sedang dikembangkan, Tiller akan dihilangkan untuk alasan keamanan dan penyederhanaan proses pemasangan charts pada kluster Kubernetes.

Agar Tiller dapat berjalan di dalam kluster Kubernetes maka perlu dibuat _serviceaccount_ dengan _role_ cluster-admin. 

```
$ kubectl -n kube-system create serviceaccount tiller
```

Lanjutkan dengan memberi role cluster-admin:
```
$ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

Kemudian pasanglah Tiller dengan perintah _helm init_:
```
helm init --service-account tiller
```

_Output_:
```
. . .

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```
Sampai disini sudah selesai proses pemasangan Helm pada kluster Kubernetes anda. Selanjutnya anda bisa mulai mencoba meng- _install_ helm charts.

### Langkah 3 - Memulai helm charts
Paket aplikasi Helm disebut _charts_. Cara paling mudah adalah dengan memasang contoh _charts_ yang disediakan oleh Helm.

Lakukan terlebih dahulu pembaruan daftar _charts_ yang tersedia:
```
helm repo update
```

Sebagai contoh, memasang aplikasi basisdata mysql dengan _charts_ mysql:
```
helm install stable/mysql
```

_Output_:
```
NAME:   angry-fox
LAST DEPLOYED: Sun Jul 14 18:55:13 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                  DATA  AGE
angry-fox-mysql-test  1     1s

==> v1/PersistentVolumeClaim
NAME             STATUS   VOLUME   CAPACITY  ACCESS MODES  STORAGECLASS  AGE
angry-fox-mysql  Pending  default  1s

==> v1/Pod(related)
NAME                              READY  STATUS   RESTARTS  AGE
angry-fox-mysql-6c45b64686-55ksv  0/2    Pending  0         1s

==> v1/Secret
NAME             TYPE    DATA  AGE
angry-fox-mysql  Opaque  2     1s

==> v1/Service
NAME             TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)   AGE
angry-fox-mysql  ClusterIP  10.0.33.9   <none>       3306/TCP  1s

==> v1beta1/Deployment
NAME             READY  UP-TO-DATE  AVAILABLE  AGE
angry-fox-mysql  0/1    0           0          1s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
angry-fox-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default angry-fox-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h angry-fox-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/angry-fox-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

Untuk melihat _release_ aplikasi yang telah dilakukan Helm, lakukan perintah _helm ls_:
```
$ helm ls
NAME            REVISION        UPDATED                         STATUS          CHART                   APP VERSION    NAMESPACE
angry-fox       1               Sun Jul 14 18:55:13 2019        DEPLOYED        mysql-0.15.0            5.7.14         default
```

Untuk melakukan _uninstall release_, gunakan perintah _helm delete_:
```
$ helm delete angry-fox
release angry-fox deleted
```

Untuk melihat status _release_:
```
LAST DEPLOYED: Sun Jul 14 18:55:13 2019
NAMESPACE: default
STATUS: DELETED
```

Selamat mencoba Helm! Dokumentasi lengkap mengenai Helm dapat anda lihat di [sini](https://helm.sh/docs/).