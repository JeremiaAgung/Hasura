# CRUD di graphQL

CRUD (Create, Read, Update, Delete) adalah operasi dasar yang dapat dilakukan pada tabel di Hasura. Berikut adalah penjelasan mengenai masing-masing operasi tersebut dalam konteks Hasura:

sumber: [Mutation](https://hasura.io/docs/latest/mutations/overview/)

Berikut adalah langkah-langkahnya:

### create tabel Users

pertama, buat tabel dengan nama `users` dengan field berikut:

![image](https://github.com/user-attachments/assets/9e0373c5-f888-4cf9-82ec-55a2659e5c12)


### insert row Users

pada perubahan data database, graphql menggunakan fungsi mutation. berikut adalah contoh query mutasi untuk insert data:


![image](https://github.com/user-attachments/assets/bfc607e7-ca51-4b6d-96c9-f37fcb840b79)

### Read Table Users

Membaca tabel di Hasura menggunakan GraphQL adalah salah satu operasi dasar yang paling sering digunakan. Berikut penjelasan tentang bagaimana Anda bisa melakukan operasi read (atau select) menggunakan GraphQL di Hasura:

```
query {
  users {
    id
    name
    email
  }
}
```
berikut hasil dari read users
![image](https://github.com/user-attachments/assets/95ab5614-5488-4ffa-a8a4-01fcd74704a6)

### Update Table Users
Untuk meng-update data dalam tabel users menggunakan Hasura GraphQL, Anda akan menggunakan operasi mutation. Berikut penjelasan detail tentang bagaimana melakukan operasi update pada tabel users:
Mutation adalah operasi di GraphQL yang digunakan untuk mengubah data (seperti insert, update, dan delete) di database.

```
mutation {
  update_users(where: {id: {_eq: 1}}, _set: {name: "Jeremia Updated"}) {
    returning {
      id
      name
      email
    }
  }
}

```
![image](https://github.com/user-attachments/assets/0ca76731-b5d4-41ca-994e-7436a8f083ff)

### Delete Users
Delete dalam Hasura GraphQL digunakan untuk menghapus data dari tabel di database. Sama seperti operasi update, operasi delete dilakukan melalui mutation di GraphQL.

Berikut adalah penjelasan detail tentang cara melakukan operasi delete pada tabel menggunakan Hasura GraphQL:

```
mutation {
  delete_users(where: {id: {_eq: 1}}) {
    returning {
      id
    }
  }
}
```


![image](https://github.com/user-attachments/assets/f5d0cf5b-ceb3-4d60-9221-8ce9b929d40a)

