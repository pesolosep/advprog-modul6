# Advance Programming Tutorial

## Reflection Notes

## Commit 1

```rust
fn handle_connection(mut stream: TcpStream) 
```
fungsi `hande_connection` yang menerima parameter `stream` yang merupakan referensi **mutable** yaitu `TCPStream`. `TcpStream` merupakan representasi koneksi jaringan melalui Transmission Control Protocol (TCP).

```rust
let buf_reader = BufReader::new(&mut stream);
```
Membuat `buf_reader` yang merupakan _struct_ dari rust untuk membaca input reference 
`stream`.

```rust
let http_request: Vec<_> = buf_reader 
    .lines() 
    .map(|result| result.unwrap()) 
    .take_while(|line| !line.is_empty()) 
    .collect();
```
`buf_reader.lines()` mengembalikan iterator hasil buffer dari TCP `stream` yang berbentuk `Result<String, std::io::Error>`, artinya bisa mengembalikan `Ok ( instance String)` jika pembacaan berhasil atau `Err( instance std::io::Error)` jika terjadi kesalahan. Lalu `.map(|result| result.unwrap())` akan mengubah iterator sebelumnya menjadi iterator `String` saja melalui `.map()`. Lalu `|result| result.unwrap()` digunakan untuk mengambil nilai `Ok` atau memberhentikan program jika mendapat nilai `Err` (panic).

`.take_while(|line| !line.is_empty())` mengumpulkan baris-baris tersebut hingga menemukan baris kosong (Untuk membaca header HTTP Request saja). Dan yang terakhir `collect` mengubah iterator string sebelumnya menjadi vektor `Vec<String>`, sehingga mendefinisikan `http_request` yang merupakan vektor string header dari HTTP request-nya.


```rust
println!("Request: {:#?}", http_request);
```
Mencetak vektor `http_request` ke konsol dengan indentasi yang tepat dan baris baru pada setiap elemen vektor sehingga 
lebih mudah membaca header dari HTTP Request.

## Commit 2
Sebelumnya `handle_connection` hanya membaca header HTTP request dan mencetaknya pada konsol.

### new `handle_connection`
![commit2.png](assets%2Fcommit2.png)
```rust
let status_line = "HTTP/1.1 200 OK";
let contents = fs::read_to_string("hello.html").unwrap();
let length = contents.len();

let response =
    format!("{status_line}\r\nContent-Length:\
      {length}\r\n\r\n{contents}");
       stream.write_all(response.as_bytes()).unwrap();
```
Setelah membaca header dari HTTP request, `handle_connection` akan mengembalikan `response` pada client user melalui koneksi TCP, 

yaitu `status_line` berisi HTTP response dengan code status 200 beserta "OK" yang menandakan permintaan berhasil diproses.
``
`Content-Length` yang menginformasikan size dari body response kepada client user,

dan yang terakhir `Contents`, static HTML file yaitu `hello.html` yang merupakan body dari HTTP response nya.

## Commit 3
### Validating request and selectively responding
![commit3.png](assets%2Fcommit3.png)

Pada gambar di atas menunjukkan bahwa sekarang sudah bisa menangani ketika ada request yang tidak sesuai. Aplikasi akan memberikan response yaitu static file `404.html`.

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```
Kode di atas menangani bad request tersebut dengan cara menambahkan statement `if` dan `else` yang mengonfirmasi hasil `request_line` sama dengan `"GET / HTTP/1.1"` sebagai conditionalnya. 
Jika endpoint yang diminta berupa `/`, yakni `GET /`, maka akan sukses mengakses web dan mengembalikan status response 200 OK dilanjutkan me-render `hello.html`. 
Sebaliknya, jika terdapat endpoint selain `/` pada awal akses, maka akan mengembalikan status response 404 NOT FOUND dilanjutkan dengan me-render `404.html`.

Kode diatas bisa dilakukan _refactoring_ agar lebih rapih dan padat. 


```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Ada banyak duplikasi pada blok `if` dan `else` pada kode sebelumnya. Kedua blok kode tersebut membaca file dan menulis isi file tersebut ke dalam stream.

Perbedaan utama hanya pada `status_line` dan `filename`. Kita bisa mengeluarkan duplikasi kode tersebut dan hanya memasukkan penentuan `status_line` dan `filename` saja didalam blok kode `if` dan `else`.

Kini,  blok `if` dan `else` hanya mengembalikan nilai yang sesuai untuk `status_line` dan `filename` dalam sebuah tuple;
kemudian kita menggunakan destrukturisasi untuk menetapkan dua nilai variabel ini menggunakan pola dalam pernyataan let.

Kode yang sebelumnya duplikat kini berada di luar blok `if` dan `else` dan menggunakan variabel `status_line` dan `filename`. Hal ini memudahkan untuk melihat perbedaan antara dua kasus tersebut, dan berarti kita hanya memiliki satu tempat untuk memperbarui kode jika kita ingin mengubah cara kerja pembacaan file dan penulisan respons.


### How to split between response? 
```rust
   let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
```
Dari konteks kode ini,
Kode tersebut membaca baris HTTP request dan menentukan respons berdasarkan conditional untuk memilih `status_line` dan `filename` yang sesuai, selanjutnya membaca isi file, lalu mengirimkan respons dengan status, panjang konten, dan isi file.

### Why Refactoring is needed

Refactoring diperlukan untuk menghilangkan duplikasi kode dan membuat kode lebih mudah dipahami, dikelola, dan diperbaiki di masa mendatang. Dalam kasus ini, ada banyak duplikasi antara blok if dan else di mana keduanya melakukan operasi yang sama, yaitu membaca file dan menulis konten file ke aliran TCP. Satu-satunya perbedaan di antara keduanya adalah status line dan nama file yang digunakan. 
Dengan memisahkan kode duplikasi diluar blok `if` dan `else`, pembacaan kode menjadi lebih mudah, dan untuk modifikasi kode hanya diperlukan sekali saja dibanding sebelumnya harus modifikasi di kedua blok kode.

## Commit 4

### Slow Simulation
Pada versi fungsi `handle_connection` yang baru, dibuat respons untuk kasus baru untuk menangani request yang mengandung path /sleep. Terdapat jeda 10 detik sebelum respons diberikan oleh server. Hal ini mensimulasikan suatu request yang memerlukan waktu untuk diproses. Ketika dibuka 2 browser windows, `127.0.0.1/sleep` dan `127.0.0.1` secara berurutan, `127.0.0.1` tidak akan diproses sebelum jeda pada `127.0.0.1/sleep` selesai.

Jika server menerima request yang membutuhkan waktu lama untuk diproses, request berikutnya harus menunggu hingga request yang lama selesai, bahkan jika request baru tersebut bisa diproses dengan cepat. Ini adalah contoh dari _single-threaded server_. Jika server menerima lebih banyak request, eksekusi sekuensial ini akan menjadi tidak optimal.

