# ภาพรวม

ส่วนนี้ให้ภาพมุมสูงของ Rand หากต้องการรายละเอียดเพิ่มเติม ข้ามไปที่ส่วนถัดไปได้เลย คือ[คู่มือ](guide.md)

## ฝั่งผู้ผลิต

สิ่งที่เชื่อมผู้ผลิตและผู้บริโภคเข้าด้วยกันคือเทรต [`RngCore`] (นิยามอยู่ใน [`rand_core`] แต่ก็ใช้งานได้จาก [`rand`] เช่นกัน)

ผู้ผลิตแบบดีเทอร์มินิสติกทุกรายควร implement [`SeedableRng`] ซึ่งเกี่ยวกับการซีด PRNG

ผู้ที่ implement [`SeedableRng`] จะได้รับการรองรับจาก [`FromEntropy`] โดยอัตโนมัติ ทำให้สร้างตัวสร้างจากแหล่งความสุ่มภายนอกได้ง่ายขึ้น สำหรับการใช้งานโดยตรงกว่า [`EntropyRng`] และ [`OsRng`] ให้ข้อมูลจากแหล่งภายนอกโดยตรง

มีอัลกอริทึม PRNG "มาตรฐาน" ให้เลือกใช้ 2 ตัว คือ [`StdRng`] และ [`SmallRng`] นอกจากนี้ยังมีให้เลือกอีกมากมายจากเครตอื่นๆ ทั้งภายในขอบเขตของโปรเจกต์นี้และภายนอก

ฟังก์ชัน [`rng()`] มอบตัวสร้างเลขสุ่มแบบเธรดโลคอลที่ซีดตัวเองอัตโนมัติและมีความปลอดภัยระดับรหัสลับให้ใช้งานอย่างสะดวก

## ฝั่งผู้บริโภค

เทรต [`Rng`] เพิ่มความสะดวกสบายขึ้นบน [`RngCore`] โดยไฮไลต์ได้แก่:

-   [`Rng::random()`] ให้ค่าสุ่มของชนิดข้อมูลใดๆ ที่รองรับการแจกแจง [`StandardUniform`]
-   [`Rng::random_range(low..high)`] ให้ค่าสุ่มแบบสม่ำเสมอภายในช่วงที่กำหนด โดยช่วงจะรวมขอบล่าง `low` และไม่รวมขอบบน `high`
-   [`Rng::random_range(low..=high)`] ให้ค่าสุ่มแบบสม่ำเสมอภายในช่วงปิดที่กำหนด โดยช่วงนี้รวมทั้งขอบล่าง `low` และขอบบน `high`
-   [`Rng::random_bool(p)`] ให้ค่า `true` ด้วยความน่าจะเป็น `p`
-   [`Rng::sample(distribution)`] ให้ค่าหนึ่งค่าจาก `distribution` ที่กำหนด
-   [`Rng::fill(dest)`] เติมข้อมูลสุ่มลงใน "สไลซ์ไบต์" ใดๆ

ฟังก์ชัน [`random()`] เป็นตัวห่อหุ้ม [`Rng::random()`] บน [`rng()`]

### การแจกแจง

โมดูล [`distr`] ดูแลการแปลงข้อมูลสุ่มให้เป็นค่าสุ่มที่มีชนิดข้อมูลและความหมาย สมาชิกสำคัญได้แก่:

-   [`Distribution<T>`] คือเทรตที่ดูแลการผลิตค่า `T` จากการแจกแจงที่ตั้งค่าไว้ ฟังก์ชันหลักคือ [`Distribution::sample`]
-   [`StandardUniform`] คือการแจกแจงที่ไม่ต้องตั้งค่าอะไร รองรับการสุ่มค่าตาม "วิธีที่คาดหวัง" สำหรับชนิดข้อมูลนั้นๆ (รองรับชนิดข้อมูลต่างๆ อย่างชัดเจน ตั้งแต่จำนวนเต็ม ทศนิยม ทูเพิล แอเรย์ ไปจนถึง `Option`)
-   [`Open01`] และ [`OpenClosed01`] ให้รูปแบบการสุ่มค่าทศนิยมในช่วง 0-1 ที่แตกต่างออกไป
-   [`Uniform`] เป็นแกนหลักเบื้องหลัง [`Rng::random_range(low..high)`] ทำให้สุ่มค่าจากช่วงที่กำหนดตามชนิดข้อมูลได้อย่างสม่ำเสมอ

ยังมีการแจกแจงอื่นๆ อีกมากมาย โปรดดูเอกสาร API

### ลำดับ

โมดูล [`seq`] รองรับ:

-   การสุ่มสมาชิกหนึ่งตัว (`choose`) หรือหลายตัว (`choose_multiple`) จากอิเทอเรเตอร์และสไลซ์
-   การสุ่มแบบถ่วงน้ำหนัก (`choose_weighted` ผ่านการแจกแจง `WeightedIndex`)
-   การสับสไลซ์

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
