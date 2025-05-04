1. Ingress

Definisi

Ingress adalah objek API di Kubernetes yang mengatur akses eksternal ke layanan (services) di dalam klaster, terutama untuk jaringan HTTP/HTTPS. Ingress menyediakan aturan untuk merutekan jaringan berdasarkan hostname atau jalur URL ke layanan tertentu.

Fungsi Utama



Rute Jaringan: Mengarahkan jaringan eksternal ke layanan berdasarkan aturan (misalnya, /graphql ke layanan Hasura).


Load Balancing: Mendistribusikan jaringan ke beberapa pod untuk memastikan ketersediaan tinggi.


SSL/TLS Termination: Menangani enkripsi/dekripsi HTTPS di level Ingress.


Virtual Hosting: Memungkinkan banyak domain/subdomain diarahkan dari satu IP publik.

Dalam Konteks Hasura


Hasura diekspos melalui Ingress untuk menerima permintaan HTTP/HTTPS dari klien eksternal.

Contoh: Ingress dapat merutekan permintaan ke graphql.example.com/v1/graphql ke layanan Hasura.

Diperlukan Ingress Controller (misalnya, NGINX, Traefik) untuk memproses aturan Ingress. Anda dapat menggunakan ingress-nginx atau controller dari penyedia cloud seperti AWS ALB.


Konfigurasi Ingress untuk Hasura harus mendukung WebSocket (untuk subscription GraphQL) dan menangani header besar.

Contoh Konfigurasi Ingress

Berikut adalah contoh YAML untuk Ingress yang mengarahkan jaringan ke layanan Hasura:

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

Pastikan Ingress Controller sudah terpasang di klaster.

Aktifkan dukungan WebSocket dengan anotasi khusus (misalnya, websocket-services untuk NGINX).

Gunakan sertifikat TLS untuk mengamankan jaringan HTTPS.

2. Load Balancer

Definisi

Load Balancer adalah tipe layanan (Service) di Kubernetes yang mengekspos layanan ke dunia eksternal melalui load balancer yang dikelola oleh penyedia infrastruktur (misalnya, AWS ELB, GCP Load Balancer). Beroperasi pada lapisan 4 (TCP/UDP) atau lapisan 7 (HTTP/HTTPS, tergantung implementasi).

Fungsi Utama

Ekspos Eksternal: Memberikan alamat IP publik yang stabil untuk mengakses layanan.

Distribusi Jaringan: Mengarahkan jaringan ke pod yang sehat di belakang layanan.

Skalabilitas: Otomatis menangani peningkatan jaringan dengan mendistribusikan ke pod tambahan.

Dalam Konteks Hasura

Load Balancer dapat digunakan untuk mengekspos layanan Hasura langsung ke internet, terutama jika Ingress tidak diperlukan atau untuk protokol non-HTTP.

Contoh: Layanan Hasura dengan tipe LoadBalancer memberikan IP eksternal langsung.

Peringatan: Menggunakan Load Balancer untuk setiap layanan bisa mahal karena setiap layanan membuat load balancer baru di penyedia cloud. Untuk Hasura, lebih efisien menggunakan Ingress dengan satu Load Balancer untuk banyak layanan.

Contoh Konfigurasi Load Balancer

Berikut adalah contoh YAML untuk layanan Hasura dengan tipe Load Balancer:

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

Load Balancer cocok untuk kasus sederhana atau ketika rute berbasis URL tidak diperlukan.

Kombinasikan dengan Ingress untuk efisiensi biaya dan fleksibilitas rute.

3. Service Account

Definisi

Service Account adalah identitas di Kubernetes yang digunakan oleh proses atau aplikasi (bukan pengguna manusia) untuk berinteraksi dengan API Kubernetes atau sumber daya lainnya di dalam klaster.

Fungsi Utama


Otentikasi: Memberikan identitas untuk pod atau aplikasi agar dapat mengakses API Kubernetes.

Otorisasi: Mengontrol izin melalui Role-Based Access Control (RBAC) untuk mengakses sumber daya tertentu.

Integrasi Eksternal: Digunakan untuk mengautentikasi aplikasi ke layanan eksternal (misalnya, database, cloud services).

Dalam Konteks Hasura

Hasura berjalan sebagai pod di Kubernetes dan sering memerlukan Service Account untuk:

Mengakses Metadata Kubernetes: Misalnya, untuk auto-discovery layanan.


Integrasi dengan Cloud: Jika Hasura mengakses database eksternal (seperti AWS RDS), Service Account dengan izin tertentu diperlukan.

RBAC untuk Keamanan: Service Account dibatasi hanya untuk mengakses sumber daya yang diperlukan (misalnya, ConfigMap atau Secret).

Contoh: Service Account untuk Hasura dapat dikonfigurasi untuk membaca Secret yang menyimpan kredensial database.

Contoh Konfigurasi Service Account dan RBAC

Berikut adalah contoh YAML untuk Service Account, Role, RoleBinding, dan Deployment Hasura:

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

# Deployment menggunakan Service Account
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

Pastikan Secret atau ConfigMap (misalnya, untuk koneksi database) tersedia dan dapat diakses oleh Service Account.

Terapkan prinsip least privilege untuk membatasi izin Service Account hanya pada yang diperlukan.

Hubungan dengan Hasura

Ingress: Merutekan jaringan HTTP/HTTPS ke endpoint Hasura (/v1/graphql, /console, dll.) dengan dukungan WebSocket untuk subscription.

Load Balancer: Alternatif untuk eksposur langsung Hasura ke eksternal, tetapi lebih mahal; biasanya digunakan bersama Ingress.


Service Account: Mengamankan pod Hasura dengan identitas dan izin untuk mengakses sumber daya Kubernetes atau layanan eksternal seperti database.
