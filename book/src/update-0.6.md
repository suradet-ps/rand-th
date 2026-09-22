# การอัปเดตเป็น 0.6

ในระหว่างรอบ 0.6 นั้น Rand ได้บ้านหลังใหม่ภายใต้โปรเจกต์
[rust-random](https://github.com/rust-random) เรารู้สึกเหมือนได้อยู่บ้านแล้ว
แต่หากคุณอยากช่วยเราตกแต่ง [โลโก้ใหม่](https://github.com/rust-random/rand/issues/278) ก็จะได้รับการขอบคุณอย่างยิ่ง!

เรายังได้บ้านหลังใหม่สำหรับเอกสารที่เน้นผู้ใช้ด้วย นั่นคือหนังสือเล่มนี้!

## PRNG

PRNG ทุกตัวใน[โมดูล PRNG เดิมของเรา](https://docs.rs/rand/0.5/rand/prng/)
ถูกย้ายไปยังเครตใหม่ เรายังเพิ่มเครตอีกหนึ่งตัวที่มีอัลกอริทึม PCG
และเครตภายนอกที่มีอัลกอริทึม Xoshiro / Xoroshiro:

-   [`rand_chacha`](https://crates.io/crates/rand_chacha)
-   [`rand_hc`](https://crates.io/crates/rand_hc)
-   [`rand_isaac`](https://crates.io/crates/rand_isaac)
-   [`rand_xorshift`](https://crates.io/crates/rand_xorshift)
-   [`rand_pcg`](https://crates.io/crates/rand_pcg)
-   [`xoshiro`](https://crates.io/crates/xoshiro)

### SmallRng

ในการอัปเดตครั้งนี้ เราเปลี่ยนอัลกอริทึมเบื้องหลัง [`SmallRng`] จาก Xorshift เป็นอัลกอริทึม PCG (ไม่ว่าจะเป็น [`Pcg64Mcg`] หรือที่เรียกว่า XSL 128/64 MCG หรือ [`Pcg32`] หรือที่เรียกว่า XSH RR 64/32 LCG ซึ่งเป็นอัลกอริทึม PCG มาตรฐาน)


## ลำดับ

[โมดูล `seq`](https://docs.rs/rand/latest/rand/seq/) ถูกเขียนใหม่ทั้งหมด
และเมธอด `choose` กับ `shuffle` ถูกนำออกจากเทรต [`Rng`]
ฟังก์ชันส่วนใหญ่ตอนนี้หาได้ในเทรต [`IteratorRandom`] และ
[`SliceRandom`]

### การเลือกแบบถ่วงน้ำหนัก

การแจกแจง [`WeightedChoice`] ตอนนี้ถูกแทนที่ด้วย
[`WeightedIndex`] ซึ่งแก้ปัญหาหลายอย่างโดยทำให้ฟังก์ชันการทำงานเป็นเจเนอริกมากขึ้น

เพื่อความสะดวก เมธอด [`SliceRandom::choose_weighted`] (และตัวแปร `_mut`) ช่วยให้ใช้ค่าที่สุ่มจาก [`WeightedIndex`] กับสไลซ์ได้โดยตรง

## ฟีเจอร์อื่นๆ

### ชนิด SIMD

ตอนนี้ Rand มีการรองรับพื้นฐานสำหรับการสร้างชนิด SIMD โดยเปิดใช้ผ่าน
ฟีเจอร์แฟล็ก `simd_support`

### ชนิด `i128` / `u128`

เนื่องจากชนิดข้อมูลเหล่านี้พร้อมใช้งานบนคอมไพเลอร์ stable แล้ว ชนิดเหล่านี้จึงได้รับการรองรับโดยอัตโนมัติ (เมื่อใช้ Rust เวอร์ชันใหม่พอ) ฟีเจอร์แฟล็ก `i128_support` ยังคงอยู่เพื่อหลีกเลี่ยงการทำให้โค้ดเดิมพัง แต่ไม่ทำอะไรอีกต่อไป


[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`Pcg32`]: https://docs.rs/rand_pcg/latest/rand_pcg/type.Pcg32.html
[`Pcg64Mcg`]: https://docs.rs/rand_pcg/latest/rand_pcg/type.Pcg64Mcg.html
[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`IteratorRandom`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html
[`SliceRandom`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html
[`WeightedChoice`]: https://docs.rs/rand/0.5/rand/distributions/struct.WeightedChoice.html
[`WeightedIndex`]: https://docs.rs/rand/latest/rand/distributions/struct.WeightedIndex.html
[`SliceRandom::choose_weighted`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_weighted
