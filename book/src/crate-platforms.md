# การรองรับแพลตฟอร์ม

ด้วยการมีส่วนร่วมจากชุมชนจำนวนมาก เครตของ Rand จึงรองรับแพลตฟอร์มที่หลากหลาย

## no_std

เมื่อใช้ `default-features = false` ทั้ง `rand` และ `rand_distr` รองรับการ build แบบ `no_std` ดู [ฟีเจอร์ทั่วไป](crate-features.html#ฟีเจอรทัวไป)

## getrandom

เครต [`getrandom`] ให้ API ระดับต่ำสำหรับเข้าถึงแหล่งเลขสุ่มเฉพาะแพลตฟอร์ม
และเป็นส่วนประกอบสำคัญของ `rand` และ `rand_core` รวมถึงไลบรารีด้านการเข้ารหัสอีกหลายตัว
มันไม่ได้ตั้งใจให้ใช้งานนอกไลบรารีระดับต่ำ

### WebAssembly

เป้าหมาย `wasm32-unknown-unknown` ไม่ตั้งสมมติฐานว่า JavaScript interface ใดพร้อมใช้งาน
ดังนั้นเครต `getrandom` จึงต้องมีการตั้งค่า ดู [การรองรับ WebAssembly](https://docs.rs/getrandom/latest/getrandom/#webassembly-support)

โปรดทราบว่าเป้าหมาย `wasm32-wasi` และ `wasm32-unknown-emscripten` ไม่มีข้อจำกัดนี้

[`getrandom`]: https://docs.rs/getrandom/
