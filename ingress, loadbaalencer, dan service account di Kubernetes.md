# Kubernetes: Memahami Ingress, Load Balancer, dan Service Account untuk Hasura

tiga konsep penting dalam Kubernetes—Ingress, Load Balancer, dan Service Account—dengan fokus pada penerapannya untuk Hasura (GraphQL Engine). 

## Daftar Isi

1. [Ingress](#1-ingress)  
2. [Load Balancer](#2-load-balancer)  
3. [Service Account](#3-service-account)  


---

## 1. Ingress

### Definisi

Ingress adalah objek API Kubernetes yang mengelola akses eksternal ke layanan (services) dalam klaster, terutama untuk jaringan HTTP/HTTPS.

### Fungsi Utama

- **Rute Jaringan**: Mengarahkan jaringan eksternal ke layanan berdasarkan hostname atau jalur URL (misalnya, `/v1/graphql` ke layanan Hasura).  
- **Load Balancing**: Mendistribusikan trafik ke pod yang sehat untuk ketersediaan tinggi.  
- **SSL/TLS Termination**: Menangani enkripsi HTTPS di level Ingress.  
- **Virtual Hosting**: Mendukung banyak domain/subdomain melalui satu IP publik.

### Dalam Konteks Hasura

- Hasura diekspos melalui Ingress untuk menangani permintaan HTTP/HTTPS dari klien eksternal.  
- Ingress dapat mengarahkan `graphql.example.com/v1/graphql` ke layanan Hasura.  
- Ingress Controller seperti **NGINX** atau **Traefik** diperlukan.  
- Harus mendukung **WebSocket** untuk GraphQL subscription dan **header besar**.

### Contoh Konfigurasi

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hasura-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/websocket-services: hasura-service
spec:
  rules:
  - host: graphql.example.com
    http:
      paths:
      - path: /v1/graphql
        pathType: Prefix
        backend:
          service:
            name: hasura-service
            port:
              number: 8080
  tls:
  - hosts:
    - graphql.example.com
    secretName: hasura-tls
```
Catatan
Pastikan Ingress Controller terpasang (misalnya ingress-nginx).

Aktifkan dukungan WebSocket dengan anotasi websocket-services.

Gunakan sertifikat TLS untuk HTTPS.

2. Load Balancer
Definisi
Load Balancer adalah tipe layanan Kubernetes yang mengekspos layanan ke eksternal melalui load balancer dari infrastruktur (AWS, GCP, dsb). Beroperasi di Layer 4 (TCP/UDP) atau Layer 7 (HTTP/HTTPS).

Fungsi Utama
Ekspos Eksternal: Menyediakan IP publik untuk akses.

Distribusi Jaringan: Mendistribusikan trafik ke pod yang sehat.

Skalabilitas: Mendukung peningkatan trafik secara otomatis.

Dalam Konteks Hasura
Cocok untuk ekspos langsung ke internet (tanpa Ingress).

Tidak ideal untuk trafik HTTP kompleks atau WebSocket.

Masing-masing LoadBalancer membuat resource baru (bisa mahal).

Contoh Konfigurasi
``yaml
apiVersion: v1
kind: Service
metadata:
  name: hasura-service
spec:
  selector:
    app: hasura
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```
```
Catatan
Cocok untuk ekspos langsung.

Kombinasi dengan Ingress lebih hemat biaya dan fleksibel.

3. Service Account
Definisi
Service Account adalah identitas Kubernetes untuk proses atau aplikasi (bukan user manusia) agar dapat berinteraksi dengan API Kubernetes atau sumber daya lain.

Fungsi Utama
Otentikasi: Memberikan identitas ke pod/aplikasi.

Otorisasi (RBAC): Mengontrol akses sumber daya.

Integrasi Eksternal: Otentikasi ke layanan seperti database.

Dalam Konteks Hasura
Digunakan untuk:

Akses metadata Kubernetes (misalnya untuk auto-discovery).

Akses ke database eksternal (misalnya AWS RDS).

Kontrol keamanan (akses ke Secret/ConfigMap).

Contoh Konfigurasi

```yaml
# Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: hasura-sa
  namespace: default

# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: hasura-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["secrets", "configmaps"]
  verbs: ["get", "list"]

# Role Binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: hasura-role-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: hasura-sa
  namespace: default
roleRef:
  kind: Role
  name: hasura-role
  apiGroup: rbac.authorization.k8s.io

# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hasura
spec:
  selector:
    matchLabels:
      app: hasura
  template:
    metadata:
      labels:
        app: hasura
    spec:
      serviceAccountName: hasura-sa
      containers:
      - name: hasura
        image: hasura/graphql-engine:latest
        ports:
        - containerPort: 8080
        env:
        - name: HASURA_GRAPHQL_DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: hasura-secrets
              key: database-url
```

Catatan
Pastikan Secret/ConfigMap tersedia sebelum digunakan.

Gunakan prinsip least privilege dalam pemberian izin.

Hubungan dengan Hasura
Ingress: Merutekan HTTP/HTTPS ke endpoint Hasura (/v1/graphql, /console) dan mendukung WebSocket.

Load Balancer: Alternatif eksposur langsung, lebih mahal, namun sederhana.

Service Account: Mengamankan Hasura dengan akses minimal ke resource yang dibutuhkan.
