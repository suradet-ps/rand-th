# เริ่มต้นใช้งาน

หากคุณยังไม่ได้ติดตั้ง โปรด[ติดตั้ง Rust](https://www.rust-lang.org/learn/get-started)

ต่อไป มาสร้างเครตใหม่และเพิ่ม rand เป็นดีเพนเดนซีกัน:
```sh
cargo new randomly
cd randomly
cargo add rand 
```

ตอนนี้ วางโค้ดต่อไปนี้ลงใน `src/main.rs`:
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

ตอนนี้ลองรันกันเลย!
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

[เครต](crates.md)อื่นๆ บางตัวถูกใช้ในคู่มือนี้ เมื่อจำเป็น คุณสามารถแก้ไขส่วน `[dependencies]` ใน `Cargo.toml` หรือใช้ `cargo add` ได้:
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
