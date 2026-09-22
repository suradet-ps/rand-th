# ฟีเจอร์ของเครต

ขอแนะนำให้ตรวจสอบ `Cargo.toml` หรือ `README.md` ของเครตเพื่อดูฟีเจอร์ต่างๆ
ตั้งแต่ `rand v0.9` เป็นต้นมา เครตของ `rust-random` ใช้เฉพาะฟีเจอร์ที่ประกาศชัดเจนเท่านั้น
(กล่าวคือ ฟีเจอร์ทั้งหมดจะอยู่ในหัวข้อ `[features]`)

เวอร์ชันรีลีสของ `Cargo.toml` ดูได้จาก `docs.rs`:

-   <https://docs.rs/crate/rand/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_core/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_distr/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/chacha20/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_xoshiro/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_pcg/latest/source/Cargo.toml.orig>

## ฟีเจอร์ทั่วไป

ฟีเจอร์ต่อไปนี้เป็นฟีเจอร์ร่วมของ `rand_core`, `rand`, `rand_distr` และอาจรวมถึงเครต RNG บางตัว:

-   `std`: เปิดใช้ฟังก์ชันที่ต้องพึ่งพาไลบรารี `std` โดยเปิดเป็นค่าเริ่มต้นยกเว้นใน `rand_core` สำหรับการใช้งานแบบ `no_std` ให้ใช้ `default-features = false`
-   `alloc`: เปิดใช้ฟังก์ชันที่ต้องใช้ allocator (สำหรับใช้งานกับ `no_std`) ฟีเจอร์นี้ถูกเปิดตามอัตโนมัติเมื่อใช้ `std`
-   `serde`: เปิดใช้การซีเรียลไลซ์ผ่าน [`serde`] เวอร์ชัน 1.0

## ฟีเจอร์ของ rand_distr

ฟังก์ชันทศนิยมจาก `num_traits` และ `libm` ถูกใช้เพื่อรองรับสภาพแวดล้อม `no_std`
และรับประกันความสามารถในการทำซ้ำ หากต้องการใช้ฟังก์ชันทศนิยมจาก `std` แทน
ซึ่งอาจให้ความแม่นยำและประสิทธิภาพที่ดีกว่า แต่อาจให้ค่าสุ่มที่แตกต่างออกไป
สามารถเปิดใช้ฟีเจอร์ `std_math` ได้ (โปรดทราบว่าเครตอื่นใดที่พึ่งพาฟีเจอร์ `std`
ของ `num-traits` (ซึ่งเปิดเป็นค่าเริ่มต้น) จะให้ผลแบบเดียวกันด้วย)

[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`serde`]: https://serde.rs/
