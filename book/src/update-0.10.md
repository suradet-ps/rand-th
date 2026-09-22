# การอัปเดตเป็น 0.10

ต่อไปนี้เป็นคำแนะนำสำหรับการพอร์ตโค้ดของคุณจาก
`rand 0.9` ไปเป็น `rand 0.10`

ต่อไปนี้เป็นคู่มือการย้ายเวอร์ชันที่เน้นการเปลี่ยนแปลงซึ่งอาจทำให้โค้ดเดิมใช้ไม่ได้ สำหรับรายการการเปลี่ยนแปลงทั้งหมด ดูบันทึกการเปลี่ยนแปลงที่เกี่ยวข้อง:

-   [CHANGELOG.md](https://github.com/rust-random/rand/blob/master/CHANGELOG.md)
-   [rand_core/CHANGELOG.md](https://docs.rs/crate/rand_core/latest/source/CHANGELOG.md)


## เทรต [Try]Rng

เทรต `RngCore, TryRngCore` ใน `rand_core` ถูกเปลี่ยนชื่อเป็น `Rng, TryRng` และเทรต `Rng` ใน `rand` ถูกเปลี่ยนชื่อเป็น `RngExt` ตามลำดับ

ความสัมพันธ์ระหว่างเทรตเหล่านี้ก็เปลี่ยนไปเช่นกัน ก่อนหน้านี้ทุก `R: RngCore` implement `TryRngCore` แต่นอกเหนือจากนั้นทั้งสองเทรตเป็นอิสระต่อกัน ตอนนี้ `Rng: TryRng<Error = Infallible>` และทุก `R: TryRng<Error = Infallible> + ?Sized` implement `Rng`

ยิ่งไปกว่านั้น แม้ก่อนหน้านี้เรา implement `R: RngCore` ให้กับทุก `R: DerefMut where R::Target: RngCore` แต่เราทำเช่นนั้นไม่ได้เนื่องจากข้อผิดพลาดเรื่องเทรตขัดแย้งกัน (การแก้ปัญหานี้ต้องใช้ specialization หรือ negative trait bound) ตอนนี้เรา implement `R: TryRng` ให้กับทุก `R: DerefMut where R::Target: TryRng` ซึ่งทำให้เกิด `R: Rng` สำหรับทุก `R: DerefMut where R::Target: Rng` ด้วย

ผลกระทบที่ใหญ่ที่สุดคือ PRNG ที่ไม่ล้มเหลวต้อง implement `TryRng` โดยมี `Error = Infallible` แทนการ implement `RngCore`

ผู้ใช้ `rand` มักต้อง import `rand::RngExt` และอาจต้องย้ายจาก `R: RngCore` ไปเป็น `R: Rng` (โปรดทราบว่าในจุดที่เคยใช้ `R: Rng` อาจเป็นการดีกว่าที่จะคง `R: Rng` ไว้ แม้ว่าการแทนที่โดยตรงจะเป็น `R: RngExt`; ข้อกำหนดขอบเขตทั้งสองเทียบเท่ากันสำหรับ `R: Sized`)


## SysRng

`rand_core::OsRng` ถูกแทนที่ด้วย `getrandom::SysRng` (พร้อมใช้งานเป็น `rand::rngs::SysRng` ด้วย)

เมธอด `SeedableRng::from_os_rng` และ `try_from_os_rng` จึงถูกนำออก [`rand::make_rng()`] ถูกจัดเตรียมไว้เป็นตัวแทนบางส่วน มิฉะนั้นให้ใช้ `SomeRng::try_from_rng(&mut SysRng).unwrap()`


## PRNG

`StdRng` ตอนนี้ให้บริการโดย `chacha20` แทน `rand_chacha` สำหรับตอนนี้ทั้งสองแพ็กเกจยังได้รับการดูแลอยู่ แต่ `rand_chacha` อาจถูกยกเลิกในอนาคต ชนิด `ChaCha{8,12,20}Rng` เป็นตัวแทนโดยตรงของชนิดที่มีชื่อเดียวกันใน `rand_chacha`; สิ่งเหล่านี้รักษาความสามารถในการทำซ้ำของเอาต์พุตและมี API คล้ายกัน

โปรดทราบว่า `rand::rngs` ตอนนี้มี PRNG ที่มีชื่อให้ใช้หลายตัว ทำให้เขียนโค้ดที่[ทำซ้ำได้](crate-reprod.md)ง่ายขึ้น: `ChaCha{8,12,20}Rng`, `Xoshiro{128,256}PlusPlus`

เครต PRNG อื่นๆ ถูกอัปเดตด้วยการเปลี่ยนแปลงเพียงเล็กน้อย (แม้ว่าสิ่งนี้อาจไม่คงอยู่ตลอดไป ดู [rngs#98](https://github.com/rust-random/rngs/issues/98)) มีเครตใหม่เพิ่มเข้ามาหนึ่งตัว: [rand_sfc](https://docs.rs/rand_sfc/latest/rand_sfc/)

### การรองรับ Clone และการซีเรียลไลซ์

`StdRng` และ `ChaCha{8,12,20}Rng` ไม่ implement `Clone` หรือเทรตของ [serde] อีกต่อไป นี่เป็นทางเลือกโดยเจตนาเพื่อป้องกันการทำสำเนาคีย์สตรีมโดยไม่ตั้งใจหรือการบันทึกไปยังที่จัดเก็บภายนอก โปรดทราบว่ายังสามารถโคลนหรือซีเรียลไลซ์ RNG เหล่านี้ได้โดยสร้างอินสแตนซ์ใหม่ด้วยคีย์เดียวกัน แล้วตั้งสายธาร (หากใช้ได้) และตำแหน่งคำ ตัวอย่างเช่น:
```rust,editable
use rand::{rngs::ChaCha8Rng, Rng, SeedableRng};

let mut rng1: ChaCha8Rng = rand::make_rng();
let _ = rng1.next_u64();

let mut rng2 = ChaCha8Rng::from_seed(rng1.get_seed());
rng2.set_stream(rng1.get_stream());
rng2.set_word_pos(rng1.get_word_pos());

assert_eq!(rng1.next_u64(), rng2.next_u64());
```


## การเปลี่ยนแปลงอื่นๆ

`TryRngCore::read_adapter` ถูกแทนที่ด้วย `rand::RngReader`

### ReseedingRng

`ReseedingRng` ถูกนำออกโดยไม่มีตัวแทน เนื่องจากเท่าที่เราสามารถตรวจสอบได้ `ThreadRng` เป็นกรณีใช้งานสำคัญเพียงกรณีเดียว เราจึงเลือกย้ายฟังก์ชันการทำงานของมันเข้าไปใน `ThreadRng` เป็นรายละเอียดของการ implement


## ดีเพนเดนซี

เครตของ Rand ตอนนี้ต้องใช้ **`rustc`** เวอร์ชัน 1.85.0 ขึ้นไป

ดีเพนเดนซีต่อ **`getrandom`** ถูกปรับขึ้นเป็นเวอร์ชัน 0.4 ดู[บันทึกการเปลี่ยนแปลงของ getrandom](https://github.com/rust-random/getrandom/blob/master/CHANGELOG.md)

### ฟีเจอร์

ฟีเจอร์แฟล็ก:

-   `os_rng` ถูกเปลี่ยนชื่อเป็น `sys_rng`
-   `small_rng` ถูกนำออก; ฟังก์ชันการทำงานของมันพร้อมใช้งานเสมอ
-   `chacha` เป็นแฟล็กใหม่ เปิดใช้งาน `rand::rngs::ChaCha{8,12,20}Rng`


## ความสามารถในการทำซ้ำ

ไม่มีการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไปของ `rand` ใน v0.10 ที่เราทราบ


[serde]: https://serde.rs/
[`rand::make_rng()`]: https://docs.rs/rand/latest/rand/fn.make_rng.html
