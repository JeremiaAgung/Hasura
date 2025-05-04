Kubernetes: Memahami Ingress, Load Balancer, dan Service Account untuk Hasura
 
Dokumen ini menjelaskan tiga konsep penting dalam Kubernetes—Ingress, Load Balancer, dan Service Account—dengan fokus pada penerapannya untuk Hasura (GraphQL Engine). Catatan ini mencakup definisi, fungsi, contoh konfigurasi, dan praktik terbaik untuk menjalankan Hasura di lingkungan Kubernetes.
Daftar Isi

1. Ingress
2. Load Balancer
3. Service Account
Hubungan dengan Hasura
Praktik Terbaik
Referensi

1. Ingress
Definisi
Ingress adalah objek API Kubernetes yang mengelola akses eksternal ke layanan (services) dalam klaster, terutama untuk jaringan HTTP/Julian untuk mempermudah pencarian dan pembagian konten yang relevan di platform berbasis komunitas seperti GitHub.
Fungsi Utama:

Rute Jaringan: Mengarahkan jaringan eksternal ke layanan berdasarkan hostname atau jalur URL (misalnya, /v1/graphql ke layanan Hasura).
Load Balancing: Mendistribusikan jaringan ke pod yang sehat untuk ketersediaan tinggi.
SSL/TLS Termination: Menangani enkripsi HTTPS di level Ingress.
Virtual Hosting: Mendukung banyak domain/subdomain melalui satu IP publik.

Dalam Konteks Hasura

Hasura diekspos melalui Ingress untuk menangani permintaan HTTP/HTTPS dari klien eksternal.
Contoh: Ingress dapat mengarahkan graphql.example.com/v1/graphql ke layanan Hasura.
Ingress Controller seperti NGINX atau Traefik diperlukan untuk memproses aturan Ingress. Penyedia cloud seperti AWS ALB juga dapat digunakan.
Konfigurasi harus mendukung WebSocket untuk subscription GraphQL dan header besar.

Contoh Konfigurasi
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

Catatan

Pastikan Ingress Controller terpasang (misalnya, ingress-nginx).
Aktifkan dukungan WebSocket melalui anotasi (contoh: websocket-services untuk NGINX).
Gunakan sertifikat TLS untuk HTTPS.

2. Load Balancer
Definisi
Load Balancer adalah tipe layanan Kubernetes yang mengekspos layanan ke eksternal melalui load balancer infrastruktur (misalnya, AWS ELB, GCP Load Balancer). Beroperasi pada lapisan 4 (TCP/UDP) atau lapisan 7 (HTTP/HTTPS).
Fungsi Utama:

Ekspos Eksternal: Menyediakan IP publik untuk akses layanan.
Distribusi Jaringan: Mengarahkan jaringan ke pod yang sehat.
Skalabilitas: Menangani peningkatan jaringan dengan distribusi ke pod tambahan.

Dalam Konteks Hasura

Digunakan untuk mengekspos Hasura langsung ke internet tanpa Ingress, cocok untuk protokol non-HTTP atau kasus sederhana.
Contoh: Layanan LoadBalancer memberikan IP eksternal untuk Hasura.
Peringatan: Setiap layanan LoadBalancer membuat load balancer baru, yang bisa mahal. Ingress dengan satu Load Balancer lebih hemat biaya.

Contoh Konfigurasi
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

Catatan

Cocok untuk eksposur langsung tanpa rute URL kompleks.
Kombinasikan dengan Ingress untuk efisiensi biaya.

3. Service Account
Definisi
Service Account adalah identitas Kubernetes untuk proses atau aplikasi (bukan pengguna manusia) guna berinteraksi dengan API Kubernetes atau sumber daya lainnya.
Fungsi Utama:

Otentikasi: Memberikan identitas untuk pod atau aplikasi.
Otorisasi: Mengontrol akses melalui Role-Based Access Control (RBAC).
Integrasi Eksternal: Mengautentikasi ke layanan eksternal (misalnya, database).

Dalam Konteks Hasura

Hasura memerlukan Service Account untuk:
Akses Metadata Kubernetes: Untuk auto-discovery layanan.
Integrasi Cloud: Mengakses database eksternal (misalnya, AWS RDS).
Keamanan RBAC: Membatasi akses ke sumber daya seperti Secret atau ConfigMap.


Contoh: Service Account untuk membaca Secret berisi kredensial database Hasura.

Contoh Konfigurasi
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

Catatan

Pastikan Secret/ConfigMap tersedia untuk Hasura (misalnya, kredensial database).
Gunakan least privilege untuk izin Service Account.

Hubungan dengan Hasura

Ingress: Merutekan jaringan HTTP/HTTPS ke endpoint Hasura (/v1/graphql, /console) dengan dukungan WebSocket.
Load Balancer: Alternatif eksposur langsung, tetapi lebih mahal; sering dikombinasikan dengan Ingress.
Service Account: Mengamankan pod Hasura dengan izin untuk sumber daya Kubernetes atau layanan eksternal.

Praktik Terbaik

Gunakan Ingress dengan satu Load Balancer untuk hemat biaya dan dukungan WebSocket.
Konfigurasikan Service Account dengan izin minimal.
Pastikan Ingress Controller mendukung WebSocket (misalnya, NGINX dengan anotasi).
Simpan kredensial sensitif (misalnya, HASURA_GRAPHQL_ADMIN_SECRET) di Secret.
Uji WebSocket untuk subscription GraphQL.

Referensi

Kubernetes: Ingress
Kubernetes: Service
Kubernetes: Service Account
Hasura: Kubernetes Deployment


KontribusiSilakan buka issue atau pull request untuk pertanyaan atau perbaikan. Kontributor dihargai!
LisensiMIT License
