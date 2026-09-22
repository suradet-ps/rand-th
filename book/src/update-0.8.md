# การอัปเดตเป็น 0.8

เนื้อหาส่วนนี้เป็นคำแนะนำสำหรับการปรับย้ายโค้ดของคุณจาก
`rand 0.7` และ `rand_distr 0.2` ไปเป็น `rand 0.8` และ `rand_distr 0.3`

## ดีเพนเดนซี

เครตในระบบของ Rand ต้องการคอมไพเลอร์ **`rustc`** เวอร์ชัน 1.36.0 ขึ้นไป
ซึ่งช่วยให้เราตัดโค้ดส่วน unsafe ออกไปได้บางส่วน และลดความซับซ้อนของตรรกะเงื่อนไข `cfg` ภายใน

ดีเพนเดนซีของ **`getrandom`** ได้รับการปรับขึ้นเป็นเวอร์ชัน 0.2 แม้ว่าสิ่งนี้จะไม่กระทบต่อ API ของ Rand โดยตรง แต่คุณอาจได้รับผลกระทบจากการเปลี่ยนแปลงที่ส่งผลต่อความเข้ากันได้ (breaking changes) บางประการ แม้ว่าจะใช้งาน `getrandom` ผ่านดีเพนเดนซีแบบสืบทอดก็ตาม:

-    คุณอาจจำเป็นต้องอัปเดตฟีเจอร์แฟล็กของ `getrandom` ที่คุณเปิดใช้งานอยู่ โดยมีฟีเจอร์ต่อไปนี้ให้ใช้งาน:
     -   `"rdrand"`: เรียกใช้คำสั่ง RDRAND บนเป้าหมาย `no_std` สถาปัตยกรรม `x86/x86_64`
     -   `"js"`: เรียกใช้ JavaScript API บนเป้าหมาย `wasm32-unknown-unknown` โดยฟีเจอร์นี้เข้ามาแทนที่ฟีเจอร์ `stdweb` และ `wasm-bindgen` ที่ถูกถอดออกไปแล้ว
     -   `"custom"`: ช่วยให้คุณสามารถระบุฟังก์ชันอิมพลีเมนต์ที่กำหนดขึ้นเองได้
-   ทาร์เก็ตแพลตฟอร์มที่ไม่ได้รับการรองรับจะไม่สามารถคอมไพล์ผ่านได้อีกต่อไป หากคุณต้องการพฤติกรรมแบบเดิม (ให้เกิด panic ในขณะรันไทม์แทนที่จะคอมไพล์ไม่ผ่าน) คุณสามารถเปิดใช้ฟีเจอร์ `custom` เพื่อให้อิมพลีเมนต์ฟังก์ชันที่มีพฤติกรรม panic ได้
-   Windows XP และ `stdweb` ไม่ได้รับการรองรับอีกต่อไปตั้งแต่ `getrandom` เวอร์ชัน 0.2.1 หากคุณจำเป็นต้องรองรับแพลตฟอร์มเหล่านี้ สามารถตรึงเวอร์ชันได้โดยเพิ่มดีเพนเดนซี `getrandom = "=0.2.0"`
-   Hermit, L4Re และ UEFI ไม่ได้รับการรองรับอย่างเป็นทางการแล้ว คุณสามารถเปิดใช้ฟีเจอร์ `rdrand` บนแพลตฟอร์มเหล่านี้แทนได้
-   เวอร์ชันเคอร์เนล Linux ขั้นต่ำที่รองรับในปัจจุบันคือ 2.6.32

หากคุณเรียกใช้งาน API ของ `getrandom` โดยตรง จะมีการเปลี่ยนแปลงที่ส่งผลกระทบต่อโค้ดเดิมเพิ่มเติม โปรดดูรายละเอียดใน [บันทึกการเปลี่ยนแปลง](https://github.com/rust-random/getrandom/blob/master/CHANGELOG.md#020---2020-09-10) ของเครตดังกล่าว

[Serde] ถูกนำกลับมาเป็นดีเพนเดนซีทางเลือกอีกครั้ง (เปิดใช้ผ่านฟีเจอร์แฟล็ก `serde1`) โดยรองรับกับประเภทข้อมูลหลายตัว (เท่าที่เหมาะสม) ทั้งนี้ได้ยกเว้น `StdRng` และ `SmallRng` ไว้อย่างตั้งใจ เนื่องจากประเภทข้อมูลเหล่านี้ไม่สามารถพอร์ตผลลัพธ์ข้ามแพลตฟอร์มได้

## ฟีเจอร์หลัก

#### `ThreadRng`

`ThreadRng` จะไม่อิมพลีเมนต์ `Copy` อีกต่อไป การเปลี่ยนแปลงนี้จำเป็นอย่างยิ่งเพื่อแก้ไขปัญหาหน่วยความจำแบบ use-after-free ที่อาจเกิดขึ้นใน destructor ของ thread-local storage โค้ดใดก็ตามที่เคยพึ่งพาการคัดลอก `ThreadRng` จะต้องปรับแก้มาส่งการอ้างอิงแบบมิวเทเบิล (mutable reference) แทน ตัวอย่างเช่น
```rust,noplayground
# use rand_0_7::distributions::{Distribution, Standard};
let rng = rand_0_7::thread_rng();
let a: u32 = Standard.sample_iter(rng).next().unwrap();
let b: u32 = Standard.sample_iter(rng).next().unwrap();
```
สามารถปรับแก้เป็นโค้ดด้านล่างนี้:
```rust,noplayground
# extern crate rand_0_8 as rand;
# use rand::prelude::*;
# use rand::distributions::Standard;
# fn main () {
let mut rng = thread_rng();
let a: u32 = Standard.sample_iter(&mut rng).next().unwrap();
let b: u32 = Standard.sample_iter(&mut rng).next().unwrap();
# }
```

#### `gen_range`

[`Rng::gen_range`] เปลี่ยนมารับพารามิเตอร์เป็นช่วง `Range` แทนการรับตัวเลข 2 ตัวแยกกัน ดังนั้นให้เปลี่ยนจาก `gen_range(a, b)` เป็น `gen_range(a..b)` เราขอแนะนำให้ใช้นิพจน์เรกิวลาร์ (regular expression) ต่อไปนี้ในการค้นหาและแทนที่ทั่วทั้งโปรเจกต์:

-   แทนที่ `gen_range\(([^,]*),\s*([^)]*)\)`
-   ด้วย `gen_range(\1..\2)`
-   หรือด้วย `gen_range($1..$2)` (หากเครื่องมือของคุณไม่รองรับ backreference)

โปรแกรมแก้ไขโค้ดหรือ IDE ส่วนใหญ่รองรับการค้นหาและแทนที่ข้ามไฟล์ หรือคุณอาจใช้เครื่องมือเฉพาะทางอย่าง Regexxer ก็ได้

การเปลี่ยนแปลงนี้ยังส่งผลสืบเนื่องอีก 2-3 ประการ:

-   รองรับช่วงแบบปิด เช่น `gen_range(1..=6)` หรือ `gen_range('A'..='Z')`
-   อาจจำเป็นต้องทำการดีเรเฟอเรนซ์ (dereference) พารามิเตอร์บางตัวอย่างชัดเจน
-   ไม่รองรับประเภทข้อมูล SIMD อีกต่อไป (แต่ยังสามารถเรียกใช้ประเภท `Uniform` ได้โดยตรง)

#### `fill`

เทรต `AsByteSliceMut` ถูกแทนที่ด้วยเทรต [`Fill`] การเปลี่ยนแปลงนี้น่าจะส่งผลกระทบเฉพาะโค้ดที่มีการอิมพลีเมนต์ `AsByteSliceMut` ให้กับประเภทข้อมูลที่ผู้ใช้สร้างขึ้นเองเท่านั้น เนื่องจาก [`Rng::fill`] และ [`Rng::try_fill`] ยังคงรองรับประเภทข้อมูลเดิมทั้งหมดตามปกติ

นอกจากนี้ `Fill` ยังรองรับประเภทสไลซ์เพิ่มเติมที่ `AsByteSliceMut` เคยทำไม่ได้ ได้แก่ `[bool], [char], [f32], [f64]`

#### `adapter`

โมดูล [`rand::rngs::adapter`] ทั้งหมดถูกจำกัดการใช้งานไว้ภายใต้ฟีเจอร์ `std` เท่านั้น แม้ว่าในทางเทคนิคจะถือเป็นการเปลี่ยนแปลงที่ส่งผลกระทบต่อโค้ดเดิม แต่ในทางปฏิบัติน่าจะกระทบเฉพาะโค้ดแบบ `no_std` ที่เรียกใช้ [`ReseedingRng`] ซึ่งแทบไม่มีการใช้งานจริงทั่วไป

## ตัวสร้าง

**StdRng** เปลี่ยนจากอัลกอริทึม ChaCha20 แบบ 20 รอบมาเป็น ChaCha12 เพื่อเพิ่มประสิทธิภาพในการทำงาน แม้จะเป็นการลดจำนวนรอบลง แต่อัลกอริทึมแบบ 12 รอบยังคงถือว่ามีความปลอดภัยสูงตามมาตรฐานวิทยาการรหัสลับ (ดู [rand#932]) ทั้งนี้ ถือเป็นการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไปสำหรับ `StdRng`

**SmallRng** เปลี่ยนมาใช้อัลกอริทึม Xoshiro128++ บนแพลตฟอร์ม 32 บิต และ Xoshiro256++ บนแพลตฟอร์ม 64 บิต ซึ่งช่วยลดความสัมพันธ์ของข้อมูลสุ่มที่สร้างจากซีดที่คล้ายกัน พร้อมทั้งช่วยเพิ่มประสิทธิภาพ การเปลี่ยนแปลงนี้ส่งผลให้ลำดับค่าสุ่มเปลี่ยนไปเช่นกัน

ในตอนนี้เราได้อิมพลีเมนต์ `PartialEq` และ `Eq` ให้กับ [`StdRng`], [`SmallRng`] และ [`StepRng`] เป็นที่เรียบร้อย

## การแจกแจง

มีการปรับเปลี่ยนย่อยหลายจุดในระบบการแจกแจงของ Rand:

-   การแจกแจง [`Uniform`] เพิ่มการรองรับประเภทข้อมูล `char` ทำให้สามารถเขียนโค้ดอย่าง `rng.gen_range('a'..='f')` ได้โดยตรง
-   เพิ่มฟังก์ชัน [`UniformSampler::sample_single_inclusive`]
-   การแจกแจง [`Alphanumeric`] เปลี่ยนมาสุ่มค่าเป็นไบต์ (`u8`) แทนที่จะเป็น `char` ซึ่งสะท้อนประเภทข้อมูลที่ประมวลผลภายในได้อย่างตรงไปตรงมามากขึ้น แต่โค้ดเดิมอาจต้องปรับแก้เพื่อแปลงจาก `u8` ไปเป็น `char` ตัวอย่างเช่น ใน Rand 0.7 คุณอาจเขียนว่า:
    ```rust,noplayground
    # use rand_0_7::{distributions::Alphanumeric, Rng};
    # let mut rng = rand_0_7::thread_rng();
    let chars: String = std::iter::repeat(())
        .map(|()| rng.sample(Alphanumeric))
        .take(7)
        .collect();
    ```
    ใน Rand 0.8 โค้ดดังกล่าวจะเทียบเท่ากับการเขียนดังนี้:
    ```rust,noplayground
    # extern crate rand_0_8 as rand;
    # use rand::{distributions::Alphanumeric, Rng};
    # fn main() {
    # let mut rng = rand::thread_rng();
    let chars: String = std::iter::repeat(())
        .map(|()| rng.sample(Alphanumeric))
        .map(char::from)
        .take(7)
        .collect();
    println!("chars = \"{chars}\"");
    # }
    ```
-   โครงสร้างทางเลือกของ [`WeightedIndex`] ที่ใช้วิธี Alias Method ถูกย้ายจากเครต `rand` ไปอยู่ที่ [`rand_distr::weighted_alias::WeightedAliasIndex`] แม้ว่าวิธี Alias Method จะเร็วกว่ามากสำหรับข้อมูลขนาดใหญ่ แต่ก็มีข้อจำกัดตรงที่การสร้างค่าเริ่มต้นค่อนข้างช้า จึงมีความหลากหลายในการใช้งานทั่วไปน้อยกว่า

ใน `rand_distr` v0.4 มีการเปลี่ยนแปลงเพิ่มเติม (นับตั้งแต่ v0.2):

-   เพิ่ม [`rand_distr::weighted_alias::WeightedAliasIndex`] เข้ามา (ย้ายมาจากเครต `rand`)
-   เพิ่มการแจกแจง [`rand_distr::InverseGaussian`] และ [`rand_distr::NormalInverseGaussian`]
-   รองรับการแจกแจงเรขาคณิต ([`Geometric`]) และการแจกแจงไฮเพอร์จีโอเมตริก ([`Hypergeometric`])
-   เปลี่ยนไปใช้อัลกอริทึมใหม่สำหรับการแจกแจงบีตา ([`Beta`]) ซึ่งช่วยปรับปรุงทั้งประสิทธิภาพและความแม่นยำ ส่งผลให้ลำดับค่าสุ่มเปลี่ยนไป
-   การแจกแจงแบบปกติ ([`Normal`]) และการแจกแจงแบบล็อกนอร์มอล ([`LogNormal`]) รองรับคอนสตรักเตอร์ `from_mean_cv` และเมธอดสุ่มตัวอย่าง `from_zscore`
-   [`rand_distr::Dirichlet`] เปลี่ยนโครงสร้างภายในมาใช้ boxed slice แทน `Vec` ดังนั้นค่าน้ำหนักจึงส่งเข้ามาเป็นสไลซ์แทน `Vec`
    ตัวอย่างเช่น โค้ดใน `rand_distr 0.2`:
    ```rust,noplayground
    # use rand_distr_0_2::Dirichlet;
    Dirichlet::new(vec![1.0, 2.0, 3.0]).unwrap();
    ```
    สามารถแทนที่ด้วยโค้ดใน `rand_distr 0.3` ดังนี้:
    ```rust,noplayground
    # use rand_distr_0_4::Dirichlet;
    Dirichlet::new(&[1.0, 2.0, 3.0]).unwrap();
    ```
-   [`rand_distr::Poisson`] ไม่รองรับการสุ่มค่า `u64` ออกมาโดยตรงอีกต่อไป โค้ดเดิมอาจต้องปรับแก้เพื่อแปลงประเภทข้อมูลจาก `f64` อย่างชัดเจน
-   เทรต `Float` ที่กำหนดขึ้นเองใน `rand_distr` ถูกแทนที่ด้วย `num_traits::Float` การอิมพลีเมนต์เทรต `Float` ให้กับประเภทข้อมูลที่ผู้ใช้สร้างขึ้นจะต้องย้ายตามไปด้วย และด้วยฟังก์ชันคณิตศาสตร์จาก `num_traits::Float` ทำให้ `rand_distr` รองรับสภาพแวดล้อม `no_std` ได้แล้ว

นอกจากนี้ยังมีการปรับปรุงเล็กๆ น้อยๆ เพิ่มเติม:

-   ปรับปรุงการจัดการความคลาดเคลื่อนจากการปัดเศษและค่า NaN สำหรับการแจกแจง [`WeightedIndex`]
-   การแจกแจง [`rand_distr::Exp`] รองรับการกำหนดค่าพารามิเตอร์ `lambda = 0` แล้ว


## ลำดับ

รองรับการสุ่มตัวอย่างแบบถ่วงน้ำหนักโดยไม่ใส่คืนแล้ว โปรดดู
[`rand::seq::index::sample_weighted`] และ
[`SliceRandom::choose_multiple_weighted`]

มีการปรับปรุงที่ส่งผลให้[ลำดับค่าสุ่มเปลี่ยนไป](https://github.com/rust-random/rand/pull/1059)
สำหรับ [`IteratorRandom::choose`] เพื่อเพิ่มความแม่นยำและประสิทธิภาพ นอกจากนี้ยังได้เพิ่ม
[`IteratorRandom::choose_stable`] เป็นอีกหนึ่งทางเลือกที่ยอมแลกประสิทธิภาพบางส่วนเพื่อไม่ให้ผลลัพธ์ขึ้นอยู่กับการคาดคะเนขนาด (size hint) ของอิเทอเรเตอร์

## ฟีเจอร์แฟล็ก

`StdRng` ถูกควบคุมผ่านฟีเจอร์แฟล็กใหม่ `std_rng` ซึ่งเปิดใช้งานไว้เป็นค่าเริ่มต้น

ฟีเจอร์ `nightly` จะไม่เปิดใช้งานฟีเจอร์ `simd_support` ให้โดยอัตโนมัติอีกต่อไป หากคุณเคยพึ่งพาฟีเจอร์นี้สำหรับการรองรับ SIMD คุณจะต้องเปิดใช้ฟีเจอร์ `simd_support` โดยตรง

## การทดสอบ

มีการเพิ่มชุดทดสอบความเสถียรของค่าผลลัพธ์ให้กับการแจกแจงทั้งหมด ([rand#786]) ซึ่งช่วยบังคับใช้กฎของเราเกี่ยวกับการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไป (ดูหัวข้อ [Reproducibility])


[`Fill`]: https://docs.rs/rand/latest/rand/trait.Fill.html
[`Rng::gen_range`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.gen_range
[`Rng::fill`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.fill
[`Rng::try_fill`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.try_fill
[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
[`StdRng`]: https://docs.rs/rand/latest/rand/rngs/struct.StdRng.html
[`StepRng`]: https://docs.rs/rand/latest/rand/rngs/mock/struct.StepRng.html
[`ThreadRng`]: https://docs.rs/rand/latest/rand/rngs/struct.ThreadRng.html
[`ReseedingRng`]: https://docs.rs/rand/latest/rand/rngs/adapter/struct.ReseedingRng.html
[`Standard`]: https://docs.rs/rand/latest/rand/distributions/struct.Standard.html
[`Uniform`]: https://docs.rs/rand/latest/rand/distributions/struct.Uniform.html
[`UniformInt`]: https://docs.rs/rand/latest/rand/distributions/struct.UniformInt.html
[`UniformSampler::sample_single_inclusive`]: https://docs.rs/rand/latest/rand/distributions/uniform/trait.UniformSampler.html#method.sample_single_inclusive
[`Alphanumeric`]: https://docs.rs/rand/latest/rand/distributions/struct.Alphanumeric.html
[`WeightedIndex`]: https://docs.rs/rand/latest/rand/distributions/struct.WeightedIndex.html
[`rand::rngs::adapter`]: https://docs.rs/rand/latest/rand/rngs/adapter/
[`rand::seq::index::sample_weighted`]: https://docs.rs/rand/latest/rand/seq/index/fn.sample_weighted.html
[`SliceRandom::choose_multiple_weighted`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#method.choose_multiple_weighted
[`IteratorRandom::choose`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html#method.choose
[`IteratorRandom::choose_stable`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html#method.choose_stable
[`rand_distr::weighted_alias::WeightedAliasIndex`]: https://docs.rs/rand_distr/latest/rand_distr/weighted_alias/struct.WeightedAliasIndex.html
[`rand_distr::InverseGaussian`]: https://docs.rs/rand_distr/latest/rand_distr/struct.InverseGaussian.html
[`rand_distr::NormalInverseGaussian`]: https://docs.rs/rand_distr/latest/rand_distr/struct.NormalInverseGaussian.html
[`rand_distr::Dirichlet`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Dirichlet.html
[`rand_distr::Poisson`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Poisson.html
[`rand_distr::Exp`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Exp.html
[`Geometric`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Geometric.html
[`Hypergeometric`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Hypergeometric.html
[`Beta`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Beta.html
[`Normal`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Normal.html
[`LogNormal`]: https://docs.rs/rand_distr/latest/rand_distr/struct.LogNormal.html
[rand#932]: https://github.com/rust-random/rand/issues/932
[rand#786]: https://github.com/rust-random/rand/issues/786
[Reproducibility]: ./crate-reprod.html
[Serde]: https://serde.rs/
