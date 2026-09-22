# ภาพรวม

ส่วนนี้สรุปภาพรวมในมุมกว้างของไลบรารี Rand หากคุณต้องการดูรายละเอียดเชิงลึก สามารถข้ามไปยังบทถัดไปได้เลยที่ [คู่มือ](guide.md)

## ฝั่งผู้ผลิต

สิ่งที่เชื่อมโยงระหว่างผู้ผลิตเลขสุ่มและส่วนการนำไปใช้งานเข้าด้วยกันคือเทรต [`RngCore`] (ซึ่งนิยามไว้ใน [`rand_core`] และสามารถเรียกใช้งานผ่านเครต [`rand`] ได้เช่นกัน)

ตัวสร้างเลขสุ่มแบบดีเทอร์มินิสติกทุกตัวควรต้องอิมพลีเมนต์เทรต [`SeedableRng`] ซึ่งจัดการเกี่ยวกับการกำหนดค่าซีดให้แก่ PRNG

ชนิดข้อมูลใดก็ตามที่อิมพลีเมนต์ [`SeedableRng`] จะได้รับการรองรับจาก [`FromEntropy`] โดยอัตโนมัติ ทำให้สร้างตัวสร้างเลขสุ่มจากแหล่งเอนโทรปีภายนอกได้อย่างสะดวก สำหรับการเรียกใช้แหล่งข้อมูลสุ่มภายนอกโดยตรง เครตมี [`EntropyRng`] และ [`OsRng`] เตรียมไว้ให้พร้อมใช้งาน

ไลบรารีมีอัลกอริทึม PRNG "มาตรฐาน" ให้เลือกใช้งาน 2 ตัว ได้แก่ [`StdRng`] และ [`SmallRng`] นอกจากนี้ยังมีตัวเลือกอีกมากมายจากเครตอื่นๆ ทั้งที่อยู่ภายในขอบเขตของโปรเจกต์นี้และเครตภายนอก

ฟังก์ชัน [`rng()`] มอบตัวสร้างเลขสุ่มระดับเธรดโลคอลที่กำหนดค่าซีดให้อัตโนมัติและปลอดภัยระดับรหัสลับ ซึ่งเรียกใช้งานได้อย่างสะดวกและรวดเร็ว

## ฝั่งผู้บริโภค

เทรต [`Rng`] ช่วยเสริมความสะดวกในการใช้งานเพิ่มเติมบน [`RngCore`] โดยมีฟังก์ชันเด่นๆ ดังนี้:

-   [`Rng::random()`] สุ่มค่าสำหรับชนิดข้อมูลใดๆ ที่รองรับการแจกแจง [`StandardUniform`]
-   [`Rng::random_range(low..high)`] สุ่มค่าแบบสม่ำเสมอภายในช่วงที่กำหนด โดยช่วงจะรวมขอบล่าง `low` และไม่รวมขอบบน `high` (ช่วงครึ่งเปิด)
-   [`Rng::random_range(low..=high)`] สุ่มค่าแบบสม่ำเสมอภายในช่วงปิดที่กำหนด โดยช่วงนี้จะรวมทั้งขอบล่าง `low` และขอบบน `high`
-   [`Rng::random_bool(p)`] คืนค่า `true` ตามค่าความน่าจะเป็น `p`
-   [`Rng::sample(distribution)`] สุ่มค่าหนึ่งค่าจาก `distribution` ที่กำหนด
-   [`Rng::fill(dest)`] เติมข้อมูลสุ่มลงใน "สไลซ์ไบต์" ใดๆ

ฟังก์ชัน [`random()`] เป็น wrapper สะดวกใช้ที่ห่อหุ้มการเรียก [`Rng::random()`] บน [`rng()`]

### การแจกแจง

โมดูล [`distr`] ทำหน้าที่แปลงข้อมูลสุ่มดิบให้กลายเป็นค่าสุ่มที่มีชนิดข้อมูลและความหมาย โดยมีองค์ประกอบสำคัญได้แก่:

-   [`Distribution<T>`] คือเทรตหลักที่ควบคุมการสร้างค่าชนิด `T` จากการแจกแจงที่กำหนด มีฟังก์ชันหัวใจสำคัญคือ [`Distribution::sample`]
-   [`StandardUniform`] คือการแจกแจงที่ไม่ต้องตั้งค่าคอนฟิกใดๆ รองรับการสุ่มค่าตาม "รูปแบบมาตรฐานที่คาดหวัง" สำหรับแต่ละชนิดข้อมูล (รองรับชนิดข้อมูลที่หลากหลายอย่างครอบคลุม ตั้งแต่จำนวนเต็ม ทศนิยม ทูเพิล อาร์เรย์ ไปจนถึง `Option`)
-   [`Open01`] และ [`OpenClosed01`] ให้รูปแบบการสุ่มค่าทศนิยมในช่วง 0 ถึง 1 ในรูปแบบช่วงเปิดและช่วงกึ่งเปิด
-   [`Uniform`] เป็นแกนหลักเบื้องหลังเมธอด [`Rng::random_range(low..high)`] ช่วยให้สุ่มค่าได้อย่างสม่ำเสมอจากช่วงที่กำหนดตามชนิดข้อมูล

ยังมีประเภทการแจกแจงอื่นๆ ให้เลือกใช้อีกมากมาย โปรดศึกษาเพิ่มเติมได้จากเอกสารอ้างอิง API

### ลำดับ

โมดูล [`seq`] รองรับการทำงานกับลำดับข้อมูล:

-   การสุ่มเลือกสมาชิกหนึ่งตัว (`choose`) หรือหลายตัว (`choose_multiple`) จากอิเทอเรเตอร์และสไลซ์
-   การสุ่มตัวอย่างแบบถ่วงน้ำหนัก (`choose_weighted` ผ่านการแจกแจง `WeightedIndex`)
-   การสับเปลี่ยนลำดับสมาชิกในสไลซ์ (shuffling a slice)

[`prelude`]: https://docs.rs/rand/latest/rand/prelude/
[`distr`]: https://docs.rs/rand/latest/rand/distr/
[`Rng::random_range(low..high)`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_range
[`Rng::random_range(low..=high)`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_range
[`random()`]: https://docs.rs/rand/latest/rand/fn.random.html
[`Rng::fill(dest)`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.fill
[`Rng::random_bool(p)`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_bool
[`Rng::random()`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random
[`Rng::shuffle`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.shuffle
[`RngCore`]: https://docs.rs/rand/latest/rand/trait.RngCore.html
[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`Rng::fill(dest)`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.fill
[`Rng::sample(distribution)`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.sample
[`SeedableRng`]: https://docs.rs/rand/latest/rand/trait.SeedableRng.html
[`seq`]: https://docs.rs/rand/latest/rand/seq/
[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`StdRng`]: https://docs.rs/rand/latest/rand/rngs/struct.StdRng.html
[`rng()`]: https://docs.rs/rand/latest/rand/fn.rng.html
[`StandardUniform`]: https://docs.rs/rand/latest/rand/distr/struct.StandardUniform.html
[`Uniform`]: https://docs.rs/rand/latest/rand/distr/struct.Uniform.html
[`rand`]: https://crates.io/crates/rand
[`rand_core`]: https://crates.io/crates/rand_core
[`FromEntropy`]: https://docs.rs/rand/latest/rand/trait.FromEntropy.html
[`EntropyRng`]: https://docs.rs/rand/latest/rand/rngs/struct.EntropyRng.html
[`Distribution<T>`]: https://docs.rs/rand/latest/rand/distr/trait.Distribution.html
[`Distribution::sample`]: https://docs.rs/rand/latest/rand/distr/trait.Distribution.html#tymethod.sample
[`Open01`]: https://docs.rs/rand/latest/rand/distr/struct.Open01.html
[`OpenClosed01`]: https://docs.rs/rand/latest/rand/distr/struct.OpenClosed01.html
[`OsRng`]: https://docs.rs/rand/latest/rand/rngs/struct.OsRng.html
