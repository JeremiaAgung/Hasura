# Kubernetes Components for Hasura

Di bawah ini adalah penjelasan mengenai beberapa komponen Kubernetes yang dapat digunakan untuk mengatur akses dan pengelolaan Hasura dalam klaster Kubernetes.

---

## 1. Ingress

### Pengertian:
Ingress adalah objek di Kubernetes yang mengatur rute HTTP(S) ke dalam klaster dari luar.  
Ia bertindak seperti reverse proxy, meneruskan permintaan ke service yang sesuai berdasarkan aturan (host, path, dll).

### Fungsi:
- Mengatur akses HTTP/HTTPS ke aplikasi dalam klaster.
- Mendukung fitur seperti TLS, host/path-based routing, dan rewrite rules.

### Contoh Penggunaan dengan Hasura:
Jika Hasura dijalankan di dalam klaster Kubernetes, Ingress dapat digunakan untuk mengarahkan trafik HTTP/HTTPS ke service Hasura.

```yaml
rules:
  - host: hasura.myapp.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: hasura
              port:
                number: 8080
```
🔁 Ingress ini akan meneruskan permintaan ke service Hasura di port 8080 jika pengguna mengakses hasura.myapp.com.

2. LoadBalancer
Pengertian:
LoadBalancer adalah tipe Service di Kubernetes yang secara otomatis membuat IP publik dan mengarahkan trafik ke Pod melalui node.

Fungsi:
Cocok untuk eksposur langsung layanan ke internet, seperti pada cloud provider (GCP, AWS, Azure).

Biasanya digunakan di lingkungan cloud, dan tidak bekerja otomatis di bare metal tanpa MetalLB atau solusi lain.

Contoh Penggunaan dengan Hasura:
Berikut adalah contoh konfigurasi service LoadBalancer untuk Hasura:
```
apiVersion: v1
kind: Service
metadata:
  name: hasura
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: hasura
```
🎯 Permintaan ke IP LoadBalancer akan diarahkan ke Hasura.

3. Service Account
Pengertian:
Service Account adalah identitas yang digunakan Pod atau aplikasi di dalam Kubernetes untuk melakukan permintaan ke API Kubernetes.

Fungsi:
Diperlukan saat Pod butuh akses ke Kubernetes API, seperti ketika memakai Hasura metadata API atau menyambung ke sumber daya lain dengan RBAC.

Contoh Penggunaan dengan Hasura:
Jika Hasura menggunakan Remote Schema atau Event Triggers yang perlu akses ke sistem lain di dalam klaster dengan autentikasi, Service Account bisa dikonfigurasi:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: hasura-sa
---
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      serviceAccountName: hasura-sa
```

🔒 Dengan Service Account, Hasura bisa mengakses layanan internal lain (misalnya microservice Python) secara aman.

🔗 Kesimpulan Hubungan dengan Hasura

Ingress	Mengatur akses HTTP(S) ke Hasura dari luar
LoadBalancer	Mengekspos Hasura secara langsung ke internet
Service Account	Memberi izin aman ke Hasura untuk akses internal API
