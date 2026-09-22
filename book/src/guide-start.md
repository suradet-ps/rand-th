# เริ่มต้นใช้งาน

หากคุณยังไม่ได้ติดตั้ง Rust ให้เริ่มต้นด้วยการ[ติดตั้ง Rust](https://www.rust-lang.org/learn/get-started) ก่อนเป็นอันดับแรก

จากนั้น มาสร้างเครตใหม่และเพิ่ม rand เข้ามาเป็นดีเพนเดนซีกัน:
```sh
cargo new randomly
cd randomly
cargo add rand 
```

ทีนี้ นำโค้ดต่อไปนี้ไปวางลงในไฟล์ `src/main.rs`:
```rust
use rand::prelude::*;

fn main() {
    let mut rng = rand::rng();

    println!("Random die roll: {}", rng.random_range(1..=6));
    println!("Random UUID: 0x{:X}", rng.random::<u128>());

    if rng.random() {
        println!("You got lucky!");
    }
}
```

เรียบร้อยแล้ว มาลองสั่งรันโปรแกรมกันเลย!
```sh
$ cargo run
   Compiling [..]
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.99s
     Running `target/debug/randomly`
Random die roll: 4
Random UUID: 0xEC3936A465339F8295EE11AB853CCDBF
You got lucky!
```

## เครตอื่นๆ

ในคู่มือเล่มนี้ยังมีการเรียกใช้[เครต](crates.md)อื่นๆ ประกอบด้วย เมื่อจำเป็นต้องใช้งาน คุณสามารถเลือกเพิ่มดีเพนเดนซีได้ทั้งการแก้ไขส่วน `[dependencies]` ในไฟล์ `Cargo.toml` ด้วยตนเอง หรือจะใช้คำสั่ง `cargo add` ก็ได้:
```sh
$ cargo add rand_distr
    Updating crates.io index
      Adding rand_distr v0.4.3 to dependencies
             Features:
             + alloc
             + std
             - serde
             - serde1
             - std_math
    Updating crates.io index
```
