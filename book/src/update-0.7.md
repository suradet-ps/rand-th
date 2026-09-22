# การอัปเดตเป็น 0.7

ตั้งแต่รีลีส 0.6 เป็นต้นมา [rust-random](https://github.com/rust-random)
ได้โลโก้และเครตใหม่: [getrandom]!

## ดีเพนเดนซี

เครตของ Rand ตอนนี้ต้องใช้ `rustc` เวอร์ชัน 1.32.0 ขึ้นไป
ซึ่งทำให้เราลบไฟล์ `build.rs` ทั้งหมดออกเพื่อให้คอมไพล์เร็วขึ้น

เครต Rand ตอนนี้มีดีเพนเดนซีโดยรวมน้อยลง แม้จะมีของใหม่เพิ่มมาบ้าง

## Getrandom

ตามที่กล่าวข้างต้น เรามีเครตใหม่: [getrandom] ซึ่งให้ API ขั้นต่ำสำหรับการเข้าถึงเอนโทรปีใหม่แบบไม่ขึ้นกับแพลตฟอร์ม สิ่งนี้แทนที่การ implement เดิมใน [`OsRng`] ซึ่งตอนนี้เป็นเพียงตัวห่อหุ้ม

## ฟีเจอร์หลัก

เทรต [`FromEntropy`] ตอนนี้ถูกนำออกแล้ว แต่ไม่ต้องกังวล เพราะเมธอด [`from_entropy`] ของมันยังคงให้การเริ่มต้นที่ง่ายดายจากบ้านใหม่ในเทรต [`SeedableRng`] (ซึ่งต้องให้ `rand_core` เปิดใช้ฟีเจอร์ `std` หรือ `getrandom`):
```rust,noplayground
# extern crate rand_0_7 as rand;
use rand::{SeedableRng, rngs::StdRng};
let mut rng = StdRng::from_entropy();
```

เมธอด [`SeedableRng::from_rng`] ตอนนี้ถือว่ามีค่าเสถียร: การ implement ควรให้ผลลัพธ์ที่พอร์ตได้

ชนิดข้อมูล [`Error`] ของ `rand_core` และ `rand` ถูกออกแบบใหม่ครั้งใหญ่; การใช้งานชนิดนี้โดยตรงน่าจะต้องปรับแก้

## PRNG

ส่วนนี้มีการเปลี่ยนแปลงน้อยกว่ารีลีสก่อนหน้า แต่ที่น่าสังเกตคือ:

-   [`rand_chacha`](https://crates.io/crates/rand_chacha) ถูกเขียนใหม่
    เพื่อประสิทธิภาพที่ดีขึ้นมาก (ผ่านคำสั่ง SIMD)
-   [`StdRng`] และ [`ThreadRng`] ตอนนี้ใช้อัลกอริทึม ChaCha นี่เป็นการเปลี่ยนแปลง
    ที่ทำให้ค่าผลลัพธ์เปลี่ยนไปสำหรับ [`StdRng`]
-   [`SmallRng`] ตอนนี้ถูกควบคุมด้วยฟีเจอร์แฟล็ก `small_rng`
-   เครต `xoshiro` ตอนนี้ชื่อ [`rand_xoshiro`](https://crates.io/crates/rand_xoshiro)
-   `rand_pcg` ตอนนี้รวม [`Pcg64`] ไว้ด้วย

## การแจกแจง

สำหรับการแจกแจงที่ใช้กันอย่างแพร่หลายที่สุด ([`Standard`] และ [`Uniform`]) ไม่มีการเปลี่ยนแปลงสำคัญ แต่สำหรับ*เกือบทั้งหมด*ของที่เหลือ...

-   เราเพิ่มเครตใหม่ [`rand_distr`] เพื่อบรรจุการแจกแจงทั้งหมด
    (รวมถึง re-export สิ่งที่ยังอยู่ใน [`rand::distributions`]) หากคุณ
    เคยใช้ `rand::distributions::Normal` ตอนนี้คุณใช้
    [`rand_distr::Normal`]
-   คอนสตรัคเตอร์ของการแจกแจงหลายตัวเปลี่ยนไปเพื่อคืนค่า `Result`
    แทนการ panic เมื่อเกิดข้อผิดพลาด
-   การแจกแจงหลายตัวตอนนี้เป็นเจเนอริกเหนือชนิดพารามิเตอร์ (ในกรณีส่วนใหญ่
    รองรับ `f32` และ `f64`) สิ่งนี้ช่วยในการใช้กับโค้ดเจเนอริก และช่วย
    ลดขนาดของการแจกแจงที่กำหนดพารามิเตอร์ ปัจจุบันอัลกอริทึมที่ซับซ้อนกว่า
    ใช้ `f64` ภายในเสมอ
-   [`Standard`] ตอนนี้สุ่มค่า [`NonZeroU*`] ได้

เรายังเพิ่มการแจกแจงอีกหลายตัว:

-   [`rand::distributions::weighted::alias_method::WeightedIndex`]
-   [`rand_distr::Pert`]
-   [`rand_distr::Triangular`]
-   [`rand_distr::UnitBall`]
-   [`rand_distr::UnitDisc`]
-   [`rand_distr::UnitSphere`] (เดิมชื่อ `rand::distributions::UnitSphereSurface`)


## ลำดับ

เพื่อช่วยเรื่องความสามารถในการพอร์ต การสุ่มค่าชนิด `usize` ทั้งหมดตอนนี้จะสุ่มค่า `u32` แทนเมื่อขอบเขตบนน้อยกว่า `u32::MAX` ซึ่งหมายความว่าการอัปเกรดเป็น 0.7 เป็นการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไปสำหรับการใช้งานฟังก์ชัน `seq` แต่หลังจากอัปเกรดเป็น 0.7 แล้ว ผลลัพธ์ควรสอดคล้องกันข้ามสถาปัตยกรรม CPU


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
