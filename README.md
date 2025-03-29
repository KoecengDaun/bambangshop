# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections



#### Reflection Publisher-1

##### **1. Apakah saya masih memerlukan interface/trait dalam BambangShop?**  
Dalam buku *Head First Design Patterns*, “Subscriber” didefinisikan sebagai interface agar berbagai objek berbeda bisa menjadi *subscriber*. Namun, di BambangShop, saya hanya memerlukan satu tipe *subscriber* yang menyimpan `url` dan menerima notifikasi melalui HTTP. **Karena fungsinya seragam**, saya bisa memakai satu *struct* biasa saja tanpa trait tambahan. Jika di masa depan diperlukan beberapa variasi *subscriber* dengan perilaku berbeda, barulah trait `Subscriber` menjadi relevan.

##### **2. Apakah penggunaan `Vec` memadai atau perlu `DashMap`?**  
Jika saya memakai `Vec`, saya harus melakukan pencarian dan penghapusan data secara linear (O(n)) serta membuat mekanisme manual untuk memastikan `url` atau `id` unik. Di sisi lain, `DashMap` memudahkan saya untuk melakukan *insert*, *lookup*, dan *delete* secara cepat (rata-rata O(1)) sambil tetap *thread-safe*.  
Untuk aplikasi yang harus menangani banyak *subscriber* secara paralel, **`DashMap` jauh lebih efisien**. Jika skenario saya sangat kecil dan tidak kritis soal kinerja, `Vec` boleh saja dipakai, tetapi pada umumnya `DashMap` adalah opsi yang aman dan praktis.

##### **3. Masih perlu `DashMap` atau cukup *Singleton*?**  
*Singleton* hanya memastikan ada satu instance global, tetapi tidak otomatis membuat data tersebut aman diakses banyak *thread* secara bersamaan. Saya tetap perlu mekanisme *thread-safe* tambahan, misalnya `Mutex`, `RwLock`, atau struktur data yang sudah mendukung *concurrency*.  
`DashMap` sudah mencakup *thread safety* dan pembagian *lock* internal (sharding), sehingga lebih efisien. Singkatnya, **saya tetap perlu struktur *thread-safe*** seperti `DashMap`, meskipun instance-nya diakses secara *singleton*.

#### Reflection Publisher-2

##### **1. Mengapa saya perlu memisahkan “Service” dan “Repository” dari Model pada pola MVC?**  
Secara prinsip, saya ingin menjaga tanggung jawab (*responsibility*) tiap komponen agar jelas. Model dalam MVC sebenarnya sudah mencakup data storage dan logika bisnis, tetapi pada praktiknya, menempatkan semua logika di satu tempat dapat membingungkan. Dengan memecahkannya:
- **Repository** berfokus pada akses data (misalnya ke database atau struktur penyimpanan).  
- **Service** menangani logika bisnis (misalnya validasi, notifikasi, panggilan ke repository).  

Pendekatan ini membuat kode saya lebih mudah dirawat, diuji, dan dikembangkan. Jika terjadi perubahan cara penyimpanan data (misalnya beralih dari `HashMap` ke database lain), saya cukup menyesuaikannya di lapisan Repository tanpa menggangu logika di Service.

##### **2. Apa yang terjadi jika saya hanya memakai Model?**  
Jika seluruh fungsi *CRUD* (tambah, hapus, dll.) dan logika bisnis (mis. notifikasi ke `Subscriber`) digabung dalam satu Model, kodenya akan menumpuk di satu tempat. Bayangkan ada tiga model berbeda – `Program`, `Subscriber`, dan `Notification` – lalu semua saling memanggil fungsi model lain. Hasilnya bisa sangat rumit: saya harus menelusuri banyak fungsi bercampur untuk sekadar mengubah satu alur bisnis. Dengan memisahkan *Service* dan *Repository*, perubahan di satu aspek tidak memicu perubahan tak terduga di bagian lain.

##### **3. Bagaimana Postman membantu saya?**  
Saya menggunakan Postman untuk menguji semua *endpoint* HTTP yang baru saya buat, seperti `POST /subscribe` atau `DELETE /product/{id}`. Postman memudahkan saya mengatur berbagai *request* dalam satu koleksi, lengkap dengan *environment variable* dan *authentication* (jika dibutuhkan). Selain itu, fitur seperti *tests* dan *documentation* bawaan Postman sangat membantu saya mengotomatisasi pengecekan respons, serta memudahkan tim memahami bagaimana cara memanggil *endpoint* tersebut. Untuk pekerjaan kelompok atau proyek di masa depan, kemampuan Postman melakukan *collection runner* juga memudahkan saya melakukan pengujian terstruktur secara cepat.

#### Reflection Publisher-3
