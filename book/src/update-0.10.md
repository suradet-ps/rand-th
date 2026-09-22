# การอัปเดตเป็น 0.10

ต่อไปนี้เป็นคำแนะนำสำหรับการพอร์ตโค้ดของคุณจาก
`rand 0.9` ไปเป็น `rand 0.10`

ต่อไปนี้เป็นคู่มือการย้ายเวอร์ชันที่เน้นการเปลี่ยนแปลงซึ่งอาจทำให้โค้ดเดิมใช้ไม่ได้ สำหรับรายการการเปลี่ยนแปลงทั้งหมด ดูบันทึกการเปลี่ยนแปลงที่เกี่ยวข้อง:

-   [CHANGELOG.md](https://github.com/rust-random/rand/blob/master/CHANGELOG.md)
-   [rand_core/CHANGELOG.md](https://docs.rs/crate/rand_core/latest/source/CHANGELOG.md)


## เทรต [Try]Rng

เทรต `RngCore, TryRngCore` ใน `rand_core` ได้รับการเปลี่ยนชื่อเป็น `Rng, TryRng` และเทรต `Rng` ใน `rand` ได้รับการเปลี่ยนชื่อเป็น `RngExt` ตามลำดับ

ความสัมพันธ์ระหว่างเทรตเหล่านี้ได้รับการปรับเปลี่ยนเช่นกัน ในอดีตทุก `R: RngCore` จะอิมพลีเมนต์ `TryRngCore` แต่นอกเหนือจากนั้นทั้งสองเทรตถือเป็นอิสระจากกัน แต่ในปัจจุบัน `Rng: TryRng<Error = Infallible>` และทุก `R: TryRng<Error = Infallible> + ?Sized` จะอิมพลีเมนต์ `Rng`

นอกจากนี้ แม้ว่าก่อนหน้านี้เราจะเคยพยายามอิมพลีเมนต์ `R: RngCore` ให้กับทุก `R: DerefMut where R::Target: RngCore` แต่ก็ไม่สามารถทำได้เนื่องจากเกิดข้อผิดพลาดเรื่องเทรตขัดแย้งกัน (conflicting-trait errors ซึ่งต้องอาศัยฟีเจอร์ specialization หรือ negative trait bounds จึงจะแก้ได้) ในเวอร์ชันนี้เราได้อิมพลีเมนต์ `R: TryRng` ให้กับทุก `R: DerefMut where R::Target: TryRng` ส่งผลให้มี `R: Rng` สำหรับทุก `R: DerefMut where R::Target: Rng` ไปด้วยโดยปริยาย

ผลกระทบที่สำคัญที่สุดคือ PRNG ที่ทำงานโดยไม่มีข้อผิดพลาด (infallible) จะต้องเปลี่ยนมาอิมพลีเมนต์ `TryRng` ที่กำหนด `Error = Infallible` แทนที่จะอิมพลีเมนต์ `RngCore`

ผู้ใช้งานเครต `rand` บ่อยครั้งจะต้องอิมพอร์ต `rand::RngExt` และอาจต้องปรับย้าย trait bound จาก `R: RngCore` ไปเป็น `R: Rng` (โปรดทราบว่าในจุดเดิมที่เคยใช้ `R: Rng` ขอแนะนำให้คง `R: Rng` ไว้ตามเดิม แม้ว่าตัวแทนโดยตรงจะเป็น `R: RngExt` ก็ตาม ทั้งนี้ bound ทั้งสองแบบมีผลเทียบเท่ากันสำหรับ `R: Sized`)


## SysRng

`rand_core::OsRng` ถูกแทนที่ด้วย `getrandom::SysRng` (และ re-export ไว้ใน `rand::rngs::SysRng` ด้วยเช่นกัน)

ด้วยเหตุนี้ เมธอด `SeedableRng::from_os_rng` และ `try_from_os_rng` จึงถูกถอดออกไป โดยมี [`rand::make_rng()`] เข้ามาทำหน้าที่ทดแทนในบางกรณี หรือมิฉะนั้นคุณสามารถเรียกใช้งาน `SomeRng::try_from_rng(&mut SysRng).unwrap()` แทนได้


## PRNG

`StdRng` เปลี่ยนมาให้บริการผ่านเครต `chacha20` แทน `rand_chacha` แม้ว่าในปัจจุบันทั้งสองแพ็กเกจจะยังคงได้รับการบำรุงรักษาอยู่ แต่ `rand_chacha` มีแนวโน้มที่จะหยุดการพัฒนาในอนาคต สำหรับประเภทข้อมูล `ChaCha{8,12,20}Rng` เป็นตัวแทนทดแทนโดยตรงของประเภทข้อมูลชื่อเดียวกันใน `rand_chacha` โดยยังคงรักษาความสามารถในการผลิตซ้ำของผลลัพธ์และมี API ที่ใกล้เคียงกัน

โปรดทราบว่าโมดูล `rand::rngs` ในปัจจุบันได้เตรียม PRNG ที่มีชื่อระบุชัดเจนไว้หลายตัว ช่วยให้เขียนโค้ดที่[ผลิตซ้ำผลลัพธ์ได้](crate-reprod.md)ง่ายขึ้น ได้แก่ `ChaCha{8,12,20}Rng` และ `Xoshiro{128,256}PlusPlus`

เครต PRNG อื่นๆ ได้รับการอัปเดตโดยมีการปรับแก้เพียงเล็กน้อย (แต่อาจมีการเปลี่ยนแปลงในระยะยาว ดู [rngs#98](https://github.com/rust-random/rngs/issues/98)) และมีเครตใหม่เพิ่มเข้ามาหนึ่งรายการ ได้แก่ [rand_sfc](https://docs.rs/rand_sfc/latest/rand_sfc/)

### การรองรับ Clone และการซีเรียลไลซ์

`StdRng` และ `ChaCha{8,12,20}Rng` จะไม่อิมพลีเมนต์ `Clone` หรือเทรตต่างๆ ของ [serde] อีกต่อไป นี่เป็นการตัดสินใจโดยเจตนาเพื่อป้องกันการทำสำเนาคีย์สตรีมโดยไม่ตั้งใจ หรือการบันทึกสถานะออกไปยังพื้นที่จัดเก็บภายนอก ทั้งนี้ คุณยังคงสามารถโคลนหรือทำ serialization ให้กับ RNG เหล่านี้ได้ โดยสร้างอินสแตนซ์ใหม่ขึ้นมาด้วยคีย์เดิม แล้วกำหนดค่าสตรีม (stream ถ้ามี) และตำแหน่งเวิร์ด (word position) เช่น:
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

`ReseedingRng` ถูกถอดออกไปโดยไม่มีตัวแทน เนื่องจากเท่าที่เราได้พิจารณา `ThreadRng` เป็นกรณีการใช้งานสำคัญเพียงกรณีเดียว เราจึงตัดสินใจย้ายฟังก์ชันการทำงานดังกล่าวเข้าไปไว้ภายใน `ThreadRng` เป็นรายละเอียดเชิงการอิมพลีเมนต์แทน


## ดีเพนเดนซี

เครตในระบบของ Rand ต้องการ **`rustc`** เวอร์ชัน 1.85.0 ขึ้นไป

ดีเพนเดนซีของ **`getrandom`** ได้รับการปรับขึ้นเป็นเวอร์ชัน 0.4 ดูรายละเอียดได้ใน [บันทึกการเปลี่ยนแปลงของ getrandom](https://github.com/rust-random/getrandom/blob/master/CHANGELOG.md)

### ฟีเจอร์

ฟีเจอร์แฟล็กของ Cargo มีการเปลี่ยนแปลงดังนี้:

-   `os_rng` เปลี่ยนชื่อเป็น `sys_rng`
-   `small_rng` ถูกถอดออก เนื่องจากฟังก์ชันการทำงานของมันพร้อมให้เรียกใช้ได้เสมออยู่แล้ว
-   `chacha` เป็นแฟล็กใหม่ สำหรับเปิดใช้งาน `rand::rngs::ChaCha{8,12,20}Rng`


## ความสามารถในการทำซ้ำ

เท่าที่เราทราบ ไม่มีการเปลี่ยนแปลงที่ส่งผลต่อค่าผลลัพธ์ใน `rand` เวอร์ชัน 0.10


[serde]: https://serde.rs/
[`rand::make_rng()`]: https://docs.rs/rand/latest/rand/fn.make_rng.html
