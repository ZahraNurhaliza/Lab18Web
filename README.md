# Lab18Web

## **Praktikum 5: Upload File Gambar**
### *Langkah-langkah Praktikum*
Upload Gambar pada Artikel
Menambahkan fungsi unggah gambar pada tambah artikel.
Buka kembali **Controller Artikel** pada project sebelumnya, sesuaikan kode pada method
**add** seperti berikut:
```php
public function add()
{
    // validasi data.
    $validation = \Config\Services::validation();
    $validation->setRules(['judul' => 'required']);
    $isDataValid = $validation->withRequest($this->request)->run();

    if ($isDataValid)
    {
        $file = $this->request->getFile('gambar');
        $file->move(ROOTPATH . 'public/gambar');

        $artikel = new ArtikelModel();
        $artikel->insert([
        'judul' => $this->request->getPost('judul'),
            'isi' => $this->request->getPost('isi'),
            'slug' => url_title($this->request->getPost('judul')),
            'gambar' => $file->getName(),
        ]);
        return redirect('admin/artikel');
    }
    $title = "Tambah Artikel";
    return view('artikel/form_add', compact('title'));
}
```

Kemudian pada file **views/artikel/form_add.php** tambahkan field input file seperti
berikut.
```php
<p>
<input type="file" name="gambar">
</p>
```

Dan sesuaikan tag form dengan menambahkan ecrypt type seperti berikut.
```php
<form action="" method="post" enctype="multipart/form-data">
```

Ujicoba file upload dengan mengakses menu tambah artikel.
![ss8](https://github.com/ZahraNurhaliza/Lab18Web/assets/115614417/e259ecac-60ff-4263-ae01-861115c52b8a)



## **Praktikum 6: AJAX**
### *Langkah-langkah Praktikum*
Membuat AJAX Controller
```php
<?php
namespace App\Controllers;
use CodeIgniter\Controller;
use CodeIgniter\HTTP\Request;
use CodeIgniter\HTTP\Response;
use App\Models\ArtikelModel;
class AjaxController extends Controller
{
    public function index()
    {
        return view('ajax/index');
    }
    public function getData()
    {
        $model = new ArtikelModel();
        $data = $model->findAll();
        // Kirim data dalam format JSON
        return $this->response->setJSON($data);
    }
    public function delete($id)
    {
        $model = new ArtikelModel();
        $data = $model->delete($id);
        $data = [
            'status' => 'OK'
        ];
        // Kirim data dalam format JSON
        return $this->response->setJSON($data);
    }
}
```
Membuat View
```php
<?= $this->include('template/header'); ?>
<h1>Data Artikel</h1>
<table class="table-data" id="artikelTable">
    <thead>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Status</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody></tbody>
</table>
<script src="<?= base_url('assets/js/jquery-3.6.0.min.js') ?>"></script>
<script>
    $(document).ready(function() {
    // Function to display a loading message while data is fetched
    function showLoadingMessage() {
    $('#artikelTable tbody').html('<tr><td colspan="4">Loading
data...</td></tr>');
    }
    // Buat fungsi load data
    function loadData() {
        showLoadingMessage(); // Display loading message initially
        
        // Lakukan request AJAX ke URL getData
        $.ajax({
            url: "<?= base_url('ajax/getData') ?>",
            method: "GET",
            dataType: "json",
            success: function(data) {
                // Tampilkan data yang diterima dari server
                var tableBody = "";
                for (var i = 0; i < data.length; i++) {
                    var row = data[i];
                    tableBody += '<tr>';
                    tableBody += '<td>' + row.id + '</td>';
                    tableBody += '<td>' + row.judul + '</td>';
                    // Add a placeholder for the "Status" column (modify as needed)
                    tableBody += '<td><span class="status">---</span></td>';
                    tableBody += '<td>';
                    // Replace with your desired actions (e.g., edit,delete)
                    tableBody += '<a href="<?= base_url('artikel/edit/')
?>' + row.id + '" class="btn btn-primary">Edit</a>';
                    tableBody += ' <a href="#" class="btn btn-danger 
btn-delete" data-id="' + row.id + '">Delete</a>';
                    tableBody += '</td>';
                    tableBody += '</tr>';
                }
                $('#artikelTable tbody').html(tableBody);
            }
        });
    }
    loadData();
    // Implement actions for buttons (e.g., delete confirmation)
    $(document).on('click', '.btn-delete', function(e) {
        e.preventDefault();
        var id = $(this).data('id');
        // Add confirmation dialog or handle deletion logic here

        // hapus data;
        if (confirm('Apakah Anda yakin ingin menghapus artikel ini?'))
{
            $.ajax({
                url: "<?= base_url('artikel/delete/') ?>" + id,
                method: "DELETE",
                success: function(data) {
                    loadData(); // Reload Datatables to reflect changes
                },
                error: function(jqXHR, textStatus, errorThrown) {
                    alert('Error deleting article: ' + textStatus + errorThrown);
                }
            });
        }
        console.log('Delete button clicked for ID:', id);
    });
});
</script>

<?= $this->include('template/footer'); ?>    
```


## **Praktikum 7: API**
### *Langkah-langkah Praktikum*
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
