# ฟีเจอร์ของเครต

ขอแนะนำให้ตรวจสอบรายละเอียดฟีเจอร์ต่างๆ ได้โดยตรงจากไฟล์ `Cargo.toml` หรือ `README.md` ของแต่ละเครต
ทั้งนี้ตั้งแต่ `rand v0.9` เป็นต้นมา เครตในเครือ `rust-random` ทั้งหมดได้เปลี่ยนมาใช้การประกาศฟีเจอร์อย่างชัดเจนเท่านั้น
(กล่าวคือ ทุกฟีเจอร์จะถูกระบุไว้อย่างครบถ้วนภายใต้ส่วน `[features]`)

คุณสามารถดูไฟล์ `Cargo.toml` ของเวอร์ชันรีลีสต่างๆ ได้บน `docs.rs`:

-   <https://docs.rs/crate/rand/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_core/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_distr/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/chacha20/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_xoshiro/latest/source/Cargo.toml.orig>
-   <https://docs.rs/crate/rand_pcg/latest/source/Cargo.toml.orig>

## ฟีเจอร์ทั่วไป

ฟีเจอร์ต่อไปนี้ถือเป็นฟีเจอร์ร่วมที่พบได้ทั่วไปใน `rand_core`, `rand`, `rand_distr` รวมถึงในบางเครตของตระกูล RNG:

-   `std`: เปิดใช้งานความสามารถที่ขึ้นอยู่กับไลบรารีมาตรฐาน (`std`) โดยจะเปิดใช้งานเป็นค่าเริ่มต้นเสมอ ยกเว้นใน `rand_core` (สำหรับการนำไปใช้งานในสภาพแวดล้อมแบบ `no_std` ให้กำหนด `default-features = false`)
-   `alloc`: เปิดใช้งานความสามารถที่จำเป็นต้องใช้ memory allocator (สำหรับการใช้งานในสภาพแวดล้อม `no_std`) โดยฟีเจอร์นี้จะถูกเปิดใช้งานโดยอัตโนมัติเมื่อเปิดใช้ `std`
-   `serde`: เปิดใช้งานความสามารถในการแปลงข้อมูล (serialization) ผ่านเครต [`serde`] เวอร์ชัน 1.0

## ฟีเจอร์ของ rand_distr

เครตเลือกใช้ฟังก์ชันเลขทศนิยมจาก `num_traits` และ `libm` เพื่อให้สามารถทำงานในสภาพแวดล้อม `no_std` ได้
ตลอดจนรับประกันความสามารถในการทำซ้ำของผลลัพธ์ (reproducibility) แต่หากคุณต้องการใช้ฟังก์ชันเลขทศนิยมจากไลบรารีมาตรฐาน `std`
ซึ่งอาจให้ความแม่นยำและประสิทธิภาพที่สูงกว่า (แต่อาจส่งผลให้ค่าสุ่มที่ได้แตกต่างไปจากเดิมเล็กน้อย)
ก็สามารถเปิดใช้งานฟีเจอร์ `std_math` ได้ (ข้อสังเกต: หากมีเครตอื่นในโปรเจกต์ของคุณที่พึ่งพาฟีเจอร์ `std`
ของ `num-traits` ซึ่งเปิดใช้งานเป็นค่าเริ่มต้นอยู่แล้ว ก็จะส่งผลให้เข้าสู่โหมดนี้ด้วยเช่นกัน)

[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`serde`]: https://serde.rs/
