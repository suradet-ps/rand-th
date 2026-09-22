# การอัปเดตเป็น 0.6

ในระหว่างรอบการพัฒนา 0.6 นั้น Rand ได้ย้ายมาอยู่ภายใต้โครงการ [rust-random](https://github.com/rust-random) อย่างเป็นทางการ พวกเรารู้สึกอบอุ่นเหมือนได้อยู่บ้าน และหากคุณต้องการร่วมตกแต่งบ้านใหม่กับเรา เรายินดีรับความช่วยเหลือด้าน [โลโก้ใหม่](https://github.com/rust-random/rand/issues/278) เป็นอย่างยิ่ง!

นอกจากนี้ เรายังได้สร้างบ้านหลังใหม่สำหรับเอกสารประกอบการใช้งานที่เน้นผู้ใช้เป็นศูนย์กลาง นั่นคือหนังสือคู่มือเล่มนี้นี่เอง!

## PRNG

PRNG ทุกตัวใน [โมดูล PRNG เดิมของเรา](https://docs.rs/rand/0.5/rand/prng/) ได้ถูกแยกย้ายไปยังเครตใหม่โดยเฉพาะ นอกจากนี้เรายังได้เพิ่มเครตสำหรับอัลกอริทึม PCG รวมถึงเครตภายนอกสำหรับอัลกอริทึมตระกูล Xoshiro / Xoroshiro:

-   [`rand_chacha`](https://crates.io/crates/rand_chacha)
-   [`rand_hc`](https://crates.io/crates/rand_hc)
-   [`rand_isaac`](https://crates.io/crates/rand_isaac)
-   [`rand_xorshift`](https://crates.io/crates/rand_xorshift)
-   [`rand_pcg`](https://crates.io/crates/rand_pcg)
-   [`xoshiro`](https://crates.io/crates/xoshiro)

### SmallRng

ในการอัปเดตครั้งนี้ เราได้สลับอัลกอริทึมเบื้องหลังของ [`SmallRng`] จากเดิมที่ใช้ Xorshift มาเป็นอัลกอริทึม PCG (ไม่ว่าจะเป็น [`Pcg64Mcg`] หรือที่รู้จักกันในชื่อ XSL 128/64 MCG หรือ [`Pcg32`] หรือที่รู้จักในชื่อ XSH RR 64/32 LCG ซึ่งเป็นอัลกอริทึม PCG มาตรฐาน)


## ลำดับ

[โมดูล `seq`](https://docs.rs/rand/latest/rand/seq/) ได้รับการเขียนขึ้นใหม่ทั้งหมด โดยได้นำเมธอด `choose` และ `shuffle` ออกจากเทรต [`Rng`] โดยฟังก์ชันการทำงานส่วนใหญ่สามารถเรียกใช้งานได้ผ่านเทรต [`IteratorRandom`] และ [`SliceRandom`]

### การเลือกแบบถ่วงน้ำหนัก

การแจกแจง [`WeightedChoice`] ถูกแทนที่ด้วย [`WeightedIndex`] ซึ่งช่วยแก้ไขข้อจำกัดหลายประการและทำให้รูปแบบการทำงานมีความเป็นเจเนอริกมากยิ่งขึ้น

เพื่อความสะดวก เมธอด [`SliceRandom::choose_weighted`] (รวมถึงตัวแปรแบบมิวเทเบิล `_mut`) ช่วยให้สามารถนำผลการสุ่มจาก [`WeightedIndex`] ไปใช้กับสไลซ์ได้โดยตรง

## ฟีเจอร์อื่นๆ

### ชนิด SIMD

ปัจจุบัน Rand มีการรองรับขั้นพื้นฐานสำหรับการสุ่มชนิด SIMD โดยเปิดใช้งานผ่านฟีเจอร์แฟล็ก `simd_support`

### ชนิด `i128` / `u128`

เนื่องจากชนิดข้อมูลเหล่านี้ได้รับการรองรับอย่างเป็นทางการบนคอมไพเลอร์รุ่น stable แล้ว จึงสามารถใช้งานได้โดยอัตโนมัติ (เมื่อใช้เวอร์ชันของ Rust ที่ใหม่เพียงพอ) ส่วนฟีเจอร์แฟล็ก `i128_support` ยังคงคงไว้เพื่อป้องกันโค้ดเดิมพัง แต่จะไม่มีผลการทำงานใดๆ อีกต่อไป


[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`Pcg32`]: https://docs.rs/rand_pcg/latest/rand_pcg/type.Pcg32.html
[`Pcg64Mcg`]: https://docs.rs/rand_pcg/latest/rand_pcg/type.Pcg64Mcg.html
[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`IteratorRandom`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html
[`SliceRandom`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html
[`WeightedChoice`]: https://docs.rs/rand/0.5/rand/distributions/struct.WeightedChoice.html
[`WeightedIndex`]: https://docs.rs/rand/latest/rand/distributions/struct.WeightedIndex.html
[`SliceRandom::choose_weighted`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_weighted
