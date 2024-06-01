# Lab18Web

## Langkah-langkah Praktikum
*Persiapan*
Periapan awal adalah mengunduh aplikasi REST Client, ada banyak aplikasi yang dapat digunakan untuk
keperluan tersebut. Salah satunya adalah Postman. `Postman` – Merupakan aplikasi yang berfungsi
sebagai REST Client, digunakan untuk testing REST API. Unduh apliasi Postman dari tautan berikut:
https://www.postman.com/downloads/

*Membuat Model*.
Pada modul sebelumnya sudah dibuat ArtikelModel, pada modul ini kita akan memanfaatkan model
tersebut agar dapat diakses melalui API.

*Membuat REST Controller*
Pada tahap ini, kita akan membuat file REST Controller yang berisi fungsi untuk menampilkan,
menambah, mengubah dan menghapus data. Masuklah ke direktori **app\Controllers** dan buatlah file
baru bernama **Post.php**. Kemudian, salin kode di bawah ini ke dalam file tersebut:

```php
<?php

namespace App\Controllers;

use CodeIgniter\RESTful\ResourceController;
use CodeIgniter\API\ResponseTrait;
use App\Models\ArtikelModel;

class Post extends ResourceController
{
    use ResponseTrait;
    // all users
    public function index()
    {
        $model = new ArtikelModel();
        $data['artikel'] = $model->orderBy('id', 'DESC')->findAll();
        return $this->respond($data);
    }
    // create
    public function create()
    {
        $model = new ArtikelModel();
        $data = [
            'judul' => $this->request->getVar('judul'),
            'isi' => $this->request->getVar('isi'),
        ];
        $model->insert($data);
        $response = [
            'status' => 201,
            'error' => null,
            'messages' => [
                'success' => 'Data artikel berhasil ditambahkan.'
            ]
        ];
        return $this->respondCreated($response);
    }
    // single user
    public function show($id = null)
    {
        $model = new ArtikelModel();
        $data = $model->where('id', $id)->first();
        if ($data) {
            return $this->respond($data);
        } else {
            return $this->failNotFound('Data tidak ditemukan.');
        }
    }
    // update
    public function update($id = null)
    {
        $model = new ArtikelModel();
        #$id = $this->request->getVar('id');
        $data = [
            'judul' => $this->request->getVar('judul'),
            'isi' => $this->request->getVar('isi'),
        ];
        $model->update($id, $data);
        $response = [
            'status' => 200,
            'error' => null,
            'messages' => [
                'success' => 'Data artikel berhasil diubah.'
            ]
        ];
        return $this->respond($response);
    }
    // delete
    public function delete($id = null)
    {
        $model = new ArtikelModel();
        $data = $model->where('id', $id)->delete($id);
        if ($data) {
            $model->delete($id);
            $response = [
                'status' => 200,
                'error' => null,
                'messages' => [
                    'success' => 'Data artikel berhasil dihapus.'
                ]
            ];
            return $this->respondDeleted($response);
        } else {
            return $this->failNotFound('Data tidak ditemukan.');
        }
    }
}
```

*Membuat Routing REST API*
Untuk mengakses REST API CodeIgniter, kita perlu mendefinisikan route-nya terlebih dulu.
Caranya, masuklah ke direktori app/Config dan bukalah file Routes.php. Tambahkan kode
di bawah ini:
`$routes->resource('post');`
Untuk mengecek route nya jalankan perintah berikut:
php spark routes
Selanjutnya akan muncul daftar route yang telah dibuat.
![ss1](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/c17e2aca-a1e5-42c3-84c1-cbbe7a975316)


*Testing REST API CodeIgniter*
Buka aplikasi postman dan pilih create new → HTTP Request
![ss2](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/ca28a5d6-962c-496c-ac59-9312505815d6)


*Menampilkan Semua Data*
Pilih method **GET** dan masukkan URL berikut:
http://localhost:8080/post
Lalu, klik **Send**. Jika hasil test menampilkan semua data artikel dari database, maka pengujian
berhasil.
![ss3](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/9d590737-53ed-4c75-8cd1-9eeda5fbec1b)


*Menampilkan Data Spesifik*
Masih menggunakan method **GET**, hanya perlu menambahkan ID artikel di belakang URL
seperti ini:
http://localhost:8080/post/2
Selanjutnya, klik **Send**. Request tersebut akan menampilkan data artikel yang memiliki ID
nomor **2** di database.
![ss4](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/1cc2e830-73a6-42b4-b800-b3a1355ab81c)


*Mengubah Data*
Untuk mengubah data, silakan ganti method menjadi **PUT**. Kemudian, masukkan URL artikel
yang ingin diubah. Misalnya, ingin mengubah data artikel dengan ID nomor 2, maka masukkan
URL berikut:
http://localhost:8080/post/2
Selanjutnya, pilih tab **Body**. Kemudian, pilih **x-www-form-uriencoded**. Masukkan nama
atribut tabel pada kolom **KEY** dan nilai data yang baru pada kolom **VALUE**. Kalau sudah,
klik **Send**.
![ss5](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/a953405b-a76f-494b-89e2-3f98728746b5)


*Menambahkan Data*
Anda perlu menggunakan method **POST** untuk menambahkan data baru ke database.
Kemudian, masukkan URL berikut:
http://localhost:8080/post
Pilih tab **Body**, lalu pilih **x-www-form-uriencoded**. Masukkan atribut tabel pada
kolom **KEY** dan nilai data baru di kolom **VALUE**. Jangan lupa, klik **Send**.
![ss6](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/aff51c45-3bbf-463d-ad76-48b8e266ce02)


*Menghapus Data*
Pilih method **DELETE** untuk menghapus data. Lalu, masukkan URL spesifik data mana yang
ingin di hapus. Misalnya, ingin menghapus data nomor 4, maka URL-nya seperti ini:
http://localhost:8080/post/7
Langsung saja klik **Send**, maka akan mendapatkan pesan bahwa data telah berhasil dihapus dari
database.
![ss7](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/f2e8f1c8-3d09-4c1a-a125-414ba99f9d04)


# SELESAI