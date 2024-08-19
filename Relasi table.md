# Relasi antar Tabel

Terdapat dua relationship yang ada pada graphql antara lain yaitu :

* object relationships (one-to-one)
* array relationships (one-to-many)

# Praktek
Untuk membuat relasi antar tabel dalam GraphQL dengan Hasura, perlu mendefinisikan skema tabel dan relasinya di database. Misalnya, untuk sebuah aplikasi product, kita bisa memiliki tabel-tabel seperti product dan product_details.

# object relationships (one-to-one)
adalah hanya satu dari masing- masing entity yang saling berhubungan atau berelasi.

query MyQuery {
  product_details {
    id
    product_id
    manufacturer
    description
  }
  products {
    id
    name
    price
    stock
  }
}

![image](https://github.com/user-attachments/assets/ed65b462-c51d-49a2-8aba-528369af1c2f)

# array relationships (one-to-many)
One-to-many terjadi ketika setiap entitas memiliki banyak hubungan dengan entitas lain, satu-ke-banyak. 
```
query MyQuery {
  product_details {
    id
    product_id
    manufacturer
    description
  }
  products {
    id
    name
    price
    stock
  }
}
```
![image](https://github.com/user-attachments/assets/456ea540-2dfe-41e5-8fd1-db1ba23a6880)
![image](https://github.com/user-attachments/assets/1cfc3e34-a4df-4f51-b79d-9fad0b21c2c0)


