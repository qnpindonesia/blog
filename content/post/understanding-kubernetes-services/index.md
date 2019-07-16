---
title: "Memahami Jenis-Jenis Service Kubernetes"
date: 2019-07-16
tags: [kubernetes, pod, node]
---

By [twahyono](mailto:twahyono@qnp.co.id)

### Apa itu Service?
Service digunakan untuk mengarahkan _traffic_ ke aplikasi yang berada di kluster. Seperti diketahui di dalam kluster, pod secara dinamis dikelola oleh Kubernetes _control plane_, pod bisa muncul dan hilang karena proses penjadwalan dan _autoscaling_ dan menyebabkan alamat IPnya yang berubah-ubah (_ephemeral_). Untuk memberikan akses yang konsisten terhadap aplikasi di sekumpulan pod tsb maka diperlukan bentuk _service_ ini. Kubernetes menyediakan 3 jenis _service_ yang bisa digunakan:

### 1. ClusterIP
ClusterIP akan memberikan satu alamat IP untuk aplikasi yang bisa di akses di lingkungan internal kluster.

{{< figure
img="clusterip-detail.png"
caption="IP statis disediakan untuk bisa diakses dari dalam kluster."
command="Resize"
options="800x" >}}

### 2. NodePort
NodePort menyediakan akses pada port yang sama di semua node yang selanjutnya _traffic_ akan diarahkan ke aplikasi yang dituju.
{{< figure
img="nodeport-detail.png"
caption="."
command="Resize"
options="800x" >}}

### 3. LoadBalancer
_Load balancer_ memberikan akses dari luar ke dalam aplikasi yang berjalan di dalam kluster dengan mekanisme _load balancing_.
{{< figure
img="loadbalancer-detail.png"
caption="IP publik pada load balancer agar aplikasi di dalam kluster bisa diakses dari luar."
command="Resize"
options="800x" >}}