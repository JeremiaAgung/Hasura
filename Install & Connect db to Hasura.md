### HASURA

**Hasura** adalah platform open-source yang menyediakan layanan backend berbasis GraphQL. Hasura memungkinkan pengembang untuk secara cepat dan mudah membangun API GraphQL dari database yang ada tanpa harus menulis kode backend yang rumit. Berikut ini adalah beberapa fitur dan konsep dasar dari Hasura:

**1. GraphQL API Otomatis**
Hasura secara otomatis menghasilkan GraphQL API berdasarkan skema database yang ada. Setiap tabel, view, dan fungsi yang ada dalam database dapat diekspos sebagai GraphQL query atau mutation tanpa perlu menulis kode API secara manual.

**2. Real-time Updates**
Hasura mendukung subscription GraphQL, yang memungkinkan klien menerima update real-time dari server setiap kali data diubah dalam database. Ini sangat berguna untuk aplikasi yang memerlukan notifikasi langsung saat data berubah.

**3. Authorization & Authentication**
Hasura memiliki sistem authorization bawaan yang fleksibel. Pengembang dapat mengonfigurasi aturan akses berbasis role dan menggunakan token JWT (JSON Web Token) untuk mengautentikasi pengguna dan menetapkan izin akses ke data.

**4. Data Federation**
Hasura dapat digabungkan dengan berbagai layanan dan API lainnya. Ini memungkinkan Anda untuk membuat skema GraphQL terpadu yang dapat mengakses data dari berbagai sumber, termasuk REST API eksternal.

**5. Migrasi & Manajemen Skema**
Hasura memungkinkan pengembang untuk mengelola migrasi database dan perubahan skema dengan mudah. Setiap perubahan pada skema database dapat di-track dan dikelola melalui migrasi.

**6. Ekosistem Plugin & Integrasi**
Hasura mendukung integrasi dengan berbagai layanan pihak ketiga, seperti serverless functions, layanan cloud, dan sistem CI/CD, yang memungkinkan pengembang untuk memperluas fungsionalitas Hasura sesuai kebutuhan.

**7. Multi-database Support**
Hasura dapat dihubungkan ke lebih dari satu database, memungkinkan pengembang untuk mengakses data dari berbagai sumber dalam satu endpoint GraphQL.
