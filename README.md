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

