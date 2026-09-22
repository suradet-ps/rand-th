# การอัปเดตเป็น 0.7

นับตั้งแต่การเปิดตัวเวอร์ชัน 0.6 โครงการ [rust-random](https://github.com/rust-random) ได้รับโลโก้ประจำโครงการและมีเครตใหม่อีกหนึ่งตัว ได้แก่ [getrandom]!

## ดีเพนเดนซี

เครตในระบบนิเวศของ Rand ในตอนนี้ต้องการ `rustc` เวอร์ชัน 1.32.0 ขึ้นไป ซึ่งช่วยให้เราสามารถตัดไฟล์ `build.rs` ออกไปได้ทั้งหมด ส่งผลให้การคอมไพล์ทำงานได้รวดเร็วยิ่งขึ้น

โดยภาพรวมแล้ว เครต Rand มีจำนวนดีเพนเดนซีลดลง แม้ว่าจะมีการเพิ่มดีเพนเดนซีใหม่เข้ามาบ้างก็ตาม

## Getrandom

ดังที่ได้กล่าวไปแล้ว เรามีเครตใหม่คือ [getrandom] ซึ่งมอบ API ขนาดเล็กกะทัดรัดสำหรับการเข้าถึงแหล่งเอนโทรปีสดใหม่โดยไม่ขึ้นกับแพลตฟอร์ม เครตนี้เข้ามาแทนที่การอิมพลีเมนต์เดิมใน [`OsRng`] ซึ่งปัจจุบันกลายเป็นเพียงตัวครอบ (wrapper) บางๆ เท่านั้น

## ฟีเจอร์หลัก

เทรต [`FromEntropy`] ถูกถอดออกไปแล้ว แต่ไม่ต้องกังวล เพราะเมธอด [`from_entropy`] ยังคงช่วยให้คุณเริ่มต้นสร้าง RNG ได้อย่างง่ายดายเช่นเดิม โดยย้ายไปสังกัดอยู่ในเทรต [`SeedableRng`] (ทั้งนี้ต้องให้ `rand_core` เปิดใช้งานฟีเจอร์ `std` หรือ `getrandom` ไว้ด้วย):
```rust,noplayground
# extern crate rand_0_7 as rand;
use rand::{SeedableRng, rngs::StdRng};
let mut rng = StdRng::from_entropy();
```

เมธอด [`SeedableRng::from_rng`] ในตอนนี้ถือว่ามีเสถียรภาพด้านค่าผลลัพธ์ (value-stable): การอิมพลีเมนต์ต่างๆ ควรให้ผลลัพธ์ที่เหมือนกันข้ามแพลตฟอร์ม (portable)

ประเภทข้อมูล [`Error`] ของ `rand_core` และ `rand` ได้รับการออกแบบใหม่ครั้งใหญ่ ทำให้การเรียกใช้งานประเภทข้อมูลนี้โดยตรงอาจจำเป็นต้องมีการปรับแก้โค้ด

## PRNG

กลุ่มนี้มีการเปลี่ยนแปลงน้อยกว่าในเวอร์ชันก่อนหน้า แต่มีจุดสำคัญที่น่าจับตามอง ได้แก่:

-   [`rand_chacha`](https://crates.io/crates/rand_chacha) ได้รับการเขียนขึ้นใหม่ทั้งหมด ส่งผลให้ประสิทธิภาพดีขึ้นอย่างมาก (โดยอาศัยคำสั่งชุด SIMD)
-   [`StdRng`] และ [`ThreadRng`] เปลี่ยนมาใช้อัลกอริทึม ChaCha ซึ่งถือเป็นการเปลี่ยนแปลงที่ส่งผลต่อค่าผลลัพธ์ (value-breaking change) สำหรับ [`StdRng`]
-   [`SmallRng`] ปัจจุบันถูกควบคุมด้วยฟีเจอร์แฟล็ก `small_rng`
-   เครต `xoshiro` เปลี่ยนชื่อเป็น [`rand_xoshiro`](https://crates.io/crates/rand_xoshiro)
-   `rand_pcg` เพิ่มประเภท [`Pcg64`] เข้ามาให้พร้อมใช้งาน

## การแจกแจง

สำหรับการแจกแจงที่นิยมใช้งานมากที่สุดอย่าง [`Standard`] และ [`Uniform`] นั้น ไม่มีการเปลี่ยนแปลงที่สำคัญ แต่สำหรับส่วนที่เหลือ*เกือบทั้งหมด*...

-   เราได้เพิ่มเครตใหม่คือ [`rand_distr`] เพื่อรวบรวมการแจกแจงทั้งหมดไว้ด้วยกัน (รวมทั้ง re-export รายการที่ยังคงอยู่ใน [`rand::distributions`] ด้วย) หากคุณเคยใช้งาน `rand::distributions::Normal` ในตอนนี้จะต้องเปลี่ยนมาเรียกใช้ [`rand_distr::Normal`] แทน
-   คอนสตรักเตอร์ของการแจกแจงหลายตัวเปลี่ยนมาคืนค่าเป็น `Result` แทนที่จะทำให้โปรแกรม panic เมื่อเกิดข้อผิดพลาด
-   การแจกแจงหลายตัวเปลี่ยนมาเป็นแบบเจเนอริกตามประเภทพารามิเตอร์ (โดยส่วนใหญ่จะรองรับทั้ง `f32` และ `f64`) ช่วยให้ใช้งานร่วมกับโค้ดแบบเจเนอริกได้สะดวกขึ้น และช่วยลดขนาดหน่วยความจำของการแจกแจงแบบมีพารามิเตอร์ลงได้ ทั้งนี้ อัลกอริทึมที่ซับซ้อนกว่าจะยังคงประมวลผลภายในด้วย `f64` เสมอ
-   [`Standard`] สามารถสุ่มค่าประเภท [`NonZeroU*`] ได้แล้ว

นอกจากนี้ เรายังได้เพิ่มการแจกแจงใหม่อีกหลายรายการ ได้แก่:

-   [`rand::distributions::weighted::alias_method::WeightedIndex`]
-   [`rand_distr::Pert`]
-   [`rand_distr::Triangular`]
-   [`rand_distr::UnitBall`]
-   [`rand_distr::UnitDisc`]
-   [`rand_distr::UnitSphere`] (เดิมใช้ชื่อว่า `rand::distributions::UnitSphereSurface`)


## ลำดับ

เพื่อเพิ่มความเข้ากันได้ข้ามแพลตฟอร์ม (portability) การสุ่มค่าประเภท `usize` ทั้งหมดจะเปลี่ยนไปสุ่มค่าเป็น `u32` แทนเมื่อขอบเขตบนมีค่าน้อยกว่า `u32::MAX` ซึ่งหมายความว่าการอัปเกรดเป็น 0.7 จะส่งผลให้ลำดับค่าสุ่มเปลี่ยนไป (value-breaking change) สำหรับฟังก์ชันการทำงานในกลุ่ม `seq` แต่ผลลัพธ์หลังการอัปเกรดเป็น 0.7 จะมีความสอดคล้องตรงกันเสมอในทุกสถาปัตยกรรม CPU


[`from_entropy`]: https://docs.rs/rand/latest/rand/trait.SeedableRng.html#method.from_entropy
[`SeedableRng::from_rng`]: https://docs.rs/rand/latest/rand/trait.SeedableRng.html#method.from_rng
[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`StdRng`]: https://docs.rs/rand/latest/rand/rngs/struct.StdRng.html
[`ThreadRng`]: https://docs.rs/rand/latest/rand/rngs/struct.ThreadRng.html
[`Pcg64`]: https://docs.rs/rand_pcg/latest/rand_pcg/type.Pcg64.html
[`rand::distributions::weighted::alias_method::WeightedIndex`]: https://docs.rs/rand/0.7/rand/distributions/weighted/alias_method/struct.WeightedIndex.html
[getrandom]: https://github.com/rust-random/getrandom
[`FromEntropy`]: https://docs.rs/rand/0.6.0/rand/trait.FromEntropy.html
[`SeedableRng`]: https://docs.rs/rand/latest/rand/trait.SeedableRng.html
[`Error`]: https://docs.rs/rand_core/latest/rand_core/struct.Error.html
[`Standard`]: https://docs.rs/rand/latest/rand/distributions/struct.Standard.html
[`Uniform`]: https://docs.rs/rand/latest/rand/distributions/struct.Uniform.html
[`rand::distributions`]: https://docs.rs/rand/latest/rand/distributions/
[`rand_distr`]: https://docs.rs/rand_distr/
[`rand_distr::Normal`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Normal.html
[`NonZeroU*`]: https://doc.rust-lang.org/std/num/https://docs.rs/rand_chacha/latest/rand_chacha/
[`rand_distr::Pert`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Pert.html
[`rand_distr::Triangular`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Triangular.html
[`rand_distr::UnitBall`]: https://docs.rs/rand_distr/latest/rand_distr/struct.UnitBall.html
[`rand_distr::UnitDisc`]: https://docs.rs/rand_distr/latest/rand_distr/struct.UnitDisc.html
[`rand_distr::UnitSphere`]: https://docs.rs/rand_distr/latest/rand_distr/struct.UnitSphere.html
[`OsRng`]: https://docs.rs/rand_core/latest/rand_core/struct.OsRng.html
