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

### Installasi Hasura via docker
Untuk saat ini saya menggunakan source berikut https://hasura.io/docs/latest/deployment/deployment-guides/docker/

**1. Menginstall Pada Docker Compose file**
Pertama Kita lakukan penginstallan Docker tersebut, dan saya melakukan penginstallan docker di ubuntu 22.04 seperti berikut:
```
sudo apt install docker-ce
```
Setelah Docker sudah terinstall maka kita lakukan seperti berikut
```
sudo systemctl status docker
```
![image](https://github.com/user-attachments/assets/4da30925-a632-4cad-9b98-2f6fb1c323b4)
Setelah sudah terinstall, Jalankan perintah berikut untuk mengunduh file docker compose dari hasura versi terbaru:
Pada `docker-compose.yaml` tersebut nantinya berisi kurang lebih seperti berikut:

```
version: "3.6"
services:
  postgres:
    image: postgres:15
    restart: always
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: postgrespassword
  graphql-engine:
    image: hasura/graphql-engine:v2.40.0
    ports:
      - "8080:8080"
    restart: always
    environment:
      ## postgres database to store Hasura metadata
      HASURA_GRAPHQL_METADATA_DATABASE_URL: postgres://postgres:postgrespassword@postgres:5432/postgres
      ## this env var can be used to add the above postgres database to Hasura as a data source. this can be removed/updated based on your needs
      PG_DATABASE_URL: postgres://postgres:postgrespassword@postgres:5432/postgres
      ## enable the console served by server
      HASURA_GRAPHQL_ENABLE_CONSOLE: "true" # set to "false" to disable console
      ## enable debugging mode. It is recommended to disable this in production
      HASURA_GRAPHQL_DEV_MODE: "true"
      HASURA_GRAPHQL_ENABLED_LOG_TYPES: startup, http-log, webhook-log, websocket-log, query-log
      ## uncomment next line to run console offline (i.e load console assets from server instead of CDN)
      # HASURA_GRAPHQL_CONSOLE_ASSETS_DIR: /srv/console-assets
      ## uncomment next line to set an admin secret
      # HASURA_GRAPHQL_ADMIN_SECRET: myadminsecretkey
      HASURA_GRAPHQL_METADATA_DEFAULTS: '{"backend_configs":{"dataconnector":{"athena":{"uri":"http://data-connector-agent:8081/api/v1/athena"},"mariadb":{"uri":"http://data-connector-agent:8081/api/v1/mariadb"},"mysql8":{"uri":"http://data-connector-agent:8081/api/v1/mysql"},"oracle":{"uri":"http://data-connector-agent:8081/api/v1/oracle"},"snowflake":{"uri":"http://data-connector-agent:8081/api/v1/snowflake"}}}}'
    depends_on:
      data-connector-agent:
        condition: service_healthy
  data-connector-agent:
    image: hasura/graphql-data-connector:v2.40.0
    restart: always
    ports:
      - 8081:8081
    environment:
      QUARKUS_LOG_LEVEL: ERROR # FATAL, ERROR, WARN, INFO, DEBUG, TRACE
      ## https://quarkus.io/guides/opentelemetry#configuration-reference
      QUARKUS_OPENTELEMETRY_ENABLED: "false"
      ## QUARKUS_OPENTELEMETRY_TRACER_EXPORTER_OTLP_ENDPOINT: http://jaeger:4317
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/api/v1/athena/health"]
      interval: 5s
      timeout: 10s
      retries: 5
      start_period: 5s
volumes:
  db_data:
```
Setelah kita sudah jalankan tampilan log pada docker seperti ini
```
$ docker-compose ps
```

![image](https://github.com/user-attachments/assets/8634bd37-2dce-4089-9ec5-17903874aeb6)




