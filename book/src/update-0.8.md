# การอัปเดตเป็น 0.8

ต่อไปนี้เป็นคำแนะนำสำหรับการพอร์ตโค้ดของคุณจาก
`rand 0.7` และ `rand_distr 0.2` ไปเป็น `rand 0.8` และ `rand_distr 0.3`

## ดีเพนเดนซี

เครตของ Rand ตอนนี้ต้องใช้ **`rustc`** เวอร์ชัน 1.36.0 ขึ้นไป
ซึ่งทำให้เราลบโค้ด unsafe บางส่วนออกและลดความซับซ้อนของตรรกะ `cfg` ภายในได้

ดีเพนเดนซีต่อ **`getrandom`** ถูกปรับขึ้นเป็นเวอร์ชัน 0.2 แม้ว่าสิ่งนี้จะไม่กระทบ API ของ Rand แต่คุณอาจได้รับผลกระทบจากการเปลี่ยนแปลงที่ทำให้โค้ดเดิมใช้ไม่ได้บางอย่าง แม้ว่าคุณจะใช้ `getrandom` เป็นเพียงดีเพนเดนซีก็ตาม:

-    คุณอาจต้องอัปเดตฟีเจอร์ของ `getrandom` ที่คุณใช้อยู่ ฟีเจอร์ต่อไปนี้
     พร้อมใช้งานแล้ว:
     -   `"rdrand"`: ใช้คำสั่ง RDRAND บนเป้าหมาย `no_std` `x86/x86_64`
     -   `"js"`: ใช้การเรียก JavaScript บน `wasm32-unknown-unknown` สิ่งนี้
         แทนที่ฟีเจอร์ `stdweb` และ `wasm-bindgen` ซึ่งถูกนำออกแล้ว
     -   `"custom"`: ให้คุณระบุการ implement ที่กำหนดเองได้
-   เป้าหมายที่ไม่รองรับจะไม่คอมไพล์อีกต่อไป หากคุณต้องการพฤติกรรมเดิม
    (panic ตอนรันไทม์แทนที่จะคอมไพล์ไม่ผ่าน) คุณสามารถใช้ฟีเจอร์
    `custom` เพื่อให้การ implement ที่ panic ได้
-   Windows XP และ stdweb ตั้งแต่ `getrandom` เวอร์ชัน 0.2.1 เป็นต้นมา
    ไม่ได้รับการรองรับแล้ว หากคุณต้องการรองรับแพลตฟอร์มใดแพลตฟอร์มหนึ่งเหล่านี้
    คุณสามารถเพิ่มดีเพนเดนซี `getrandom = "=0.2.0"` เพื่อปักหมุดเวอร์ชันนี้ได้
-   Hermit, L4Re และ UEFI ไม่ได้รับการรองรับอย่างเป็นทางการแล้ว คุณสามารถใช้
    ฟีเจอร์ `rdrand` บนแพลตฟอร์มเหล่านี้ได้
-   เวอร์ชันเคอร์เนล Linux ขั้นต่ำที่รองรับตอนนี้คือ 2.6.32

หากคุณใช้ API ของ `getrandom` โดยตรง มีการเปลี่ยนแปลงที่ทำให้โค้ดเดิมใช้ไม่ได้เพิ่มเติมที่อาจกระทบคุณ ดู
[บันทึกการเปลี่ยนแปลง](https://github.com/rust-random/getrandom/blob/master/CHANGELOG.md#020---2020-09-10) ของมัน

[Serde] ถูกเพิ่มกลับเป็นดีเพนเดนซีแบบเลือกได้ (ใช้ฟีเจอร์แฟล็ก `serde1`)
โดยรองรับชนิดข้อมูลหลายอย่าง (ในจุดที่เหมาะสม) `StdRng` และ `SmallRng` ถูกยกเว้นโดยเจตนา เนื่องจากชนิดข้อมูลเหล่านี้พอร์ตไม่ได้

## ฟีเจอร์หลัก

#### `ThreadRng`

`ThreadRng` ไม่ implement `Copy` อีกต่อไป สิ่งนี้จำเป็นเพื่อแก้ปัญหา use-after-free ที่อาจเกิดขึ้นในเดสทรักเตอร์แบบเธรดโลคัลของมัน โค้ดใดที่พึ่งพาการคัดลอก `ThreadRng` ต้องแก้ไขให้ใช้ mutable reference แทน ตัวอย่างเช่น
```rust,noplayground
# use rand_0_7::distributions::{Distribution, Standard};
let rng = rand_0_7::thread_rng();
let a: u32 = Standard.sample_iter(rng).next().unwrap();
let b: u32 = Standard.sample_iter(rng).next().unwrap();
```
สามารถแทนที่ด้วยโค้ดต่อไปนี้:
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

[`Rng::gen_range`] ตอนนี้รับ `Range` แทนตัวเลขสองตัว ดังนั้นให้แทนที่
`gen_range(a, b)` ด้วย `gen_range(a..b)` เราแนะนำให้ใช้ regular
expression ต่อไปนี้เพื่อค้นหาและแทนที่ในทุกไฟล์:

-   แทนที่ `gen_range\(([^,]*),\s*([^)]*)\)`
-   ด้วย `gen_range(\1..\2)`
-   หรือด้วย `gen_range($1..$2)` (หากเครื่องมือของคุณไม่รองรับ backreference)

IDE ส่วนใหญ่รองรับการค้นหาและแทนที่ข้ามไฟล์หรือคล้ายกัน หรืออาจใช้เครื่องมือภายนอกอย่าง Regexxer ก็ได้

การเปลี่ยนแปลงนี้มีผลอื่นๆ อีกสองสามประการ:

-   ตอนนี้รองรับช่วงปิด เช่น `gen_range(1..=6)` หรือ `gen_range('A'..='Z')`
-   อาจจำเป็นต้อง dereference พารามิเตอร์บางตัวอย่างชัดเจน
-   ชนิด SIMD ไม่ได้รับการรองรับอีกต่อไป (ชนิด `Uniform` ยังใช้ได้โดยตรง)

#### `fill`

เทรต `AsByteSliceMut` ถูกแทนที่ด้วยเทรต [`Fill`] สิ่งนี้ควรกระทบเฉพาะโค้ดที่ implement `AsByteSliceMut` ให้กับชนิดข้อมูลที่ผู้ใช้กำหนด เนื่องจาก
[`Rng::fill`] และ [`Rng::try_fill`] ยังคงรองรับชนิดข้อมูลที่เคยรองรับ

`Fill` รองรับชนิดสไลซ์เพิ่มเติมบางอย่างที่ `AsByteSliceMut` รองรับไม่ได้: `[bool], [char], [f32], [f64]`

#### `adapter`

โมดูล [`rand::rngs::adapter`] ทั้งหมดตอนนี้ถูกจำกัดให้ใช้ได้เฉพาะกับฟีเจอร์ `std` แม้ว่านี่จะเป็นความเปลี่ยนแปลงที่ทำให้โค้ดเดิมใช้ไม่ได้ในทางเทคนิค แต่ควรกระทบเฉพาะโค้ด `no_std` ที่ใช้ [`ReseedingRng`] ซึ่งไม่น่าจะมีอยู่จริงในโลกภายนอก

## ตัวสร้าง

**StdRng** เปลี่ยนจาก ChaCha20 แบบ 20 รอบเป็น ChaCha12 เพื่อประสิทธิภาพที่ดีขึ้น นี่เป็นการลดความซับซ้อน แต่ตัวแปรแบบ 12 รอบยังถือว่าปลอดภัย ดู [rand#932] นี่เป็นการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไปสำหรับ `StdRng`

**SmallRng** ตอนนี้ใช้อัลกอริทึม Xoshiro128++ และ Xoshiro256++ บนแพลตฟอร์ม 32 บิตและ 64 บิตตามลำดับ สิ่งนี้ลดสหสัมพันธ์ของข้อมูลสุ่มที่สร้างจากซีดที่คล้ายกันและปรับปรุงประสิทธิภาพ เป็นการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไป

ตอนนี้เรา implement `PartialEq` และ `Eq` ให้กับ [`StdRng`], [`SmallRng`] และ [`StepRng`]

## การแจกแจง

มีการเปลี่ยนแปลงเล็กๆ หลายอย่างเกิดขึ้นกับการแจกแจงของ rand:

-   การแจกแจง [`Uniform`] ตอนนี้รองรับชนิด `char` เพิ่มเติม ดังนั้น
    ตัวอย่างเช่น `rng.gen_range('a'..='f')` จึงรองรับแล้ว
-   [`UniformSampler::sample_single_inclusive`] ถูกเพิ่มเข้ามา
-   การแจกแจง [`Alphanumeric`] ตอนนี้สุ่มเป็นไบต์แทนที่จะเป็น char สิ่งนี้
    สะท้อนชนิดข้อมูลที่ใช้ภายในได้ใกล้เคียงกว่า แต่โค้ดเดิมคงต้อง
    ปรับแก้เพื่อแปลงจาก `u8` เป็น `char` ตัวอย่างเช่น กับ
    Rand 0.7 คุณเขียนได้ว่า:
    ```rust,noplayground
    # use rand_0_7::{distributions::Alphanumeric, Rng};
    # let mut rng = rand_0_7::thread_rng();
    let chars: String = std::iter::repeat(())
        .map(|()| rng.sample(Alphanumeric))
        .take(7)
        .collect();
    ```
    กับ Rand 0.8 โค้ดนี้เทียบเท่ากับต่อไปนี้:
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
-   การ implement ทางเลือกของ [`WeightedIndex`] ที่ใช้วิธี alias
    ถูกย้ายจาก `rand` ไปเป็น [`rand_distr::weighted_alias::WeightedAliasIndex`] วิธี
    alias เร็วกว่าสำหรับขนาดใหญ่ แต่มีจุดอ่อนเรื่องการเริ่มต้นที่ช้า
    จึงมีประโยชน์ทั่วไปน้อยกว่า

ใน `rand_distr` v0.4 มีการเปลี่ยนแปลงเพิ่มเติม (ตั้งแต่ v0.2):

-   [`rand_distr::weighted_alias::WeightedAliasIndex`] ถูกเพิ่มเข้ามา (ย้ายมาจากเครต `rand`)
-   [`rand_distr::InverseGaussian`] และ [`rand_distr::NormalInverseGaussian`]
    ถูกเพิ่มเข้ามา
-   การแจกแจง [`Geometric`] และ [`Hypergeometric`] ได้รับการรองรับแล้ว
-   มีการใช้อัลกอริทึมที่แตกต่างสำหรับการแจกแจง [`Beta`] ซึ่งปรับปรุงทั้ง
    ประสิทธิภาพและความแม่นยำ นี่เป็นการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไป
-   การแจกแจง [`Normal`] และ [`LogNormal`] ตอนนี้รองรับเมธอดคอนสตรัคเตอร์
    `from_mean_cv` และเมธอดสุ่ม `from_zscore`
-   [`rand_distr::Dirichlet`] ตอนนี้ใช้ boxed slice ภายในแทน `Vec`
    ดังนั้นน้ำหนักจึงถูกรับเป็นสไลซ์แทน `Vec` เป็นอินพุต
    ตัวอย่างเช่น โค้ด `rand_distr 0.2` ต่อไปนี้
    ```rust,noplayground
    # use rand_distr_0_2::Dirichlet;
    Dirichlet::new(vec![1.0, 2.0, 3.0]).unwrap();
    ```
    สามารถแทนที่ด้วยโค้ด `rand_distr 0.3` ต่อไปนี้:
    ```rust,noplayground
    # use rand_distr_0_4::Dirichlet;
    Dirichlet::new(&[1.0, 2.0, 3.0]).unwrap();
    ```
-   [`rand_distr::Poisson`] ไม่รองรับการสุ่มค่า `u64` โดยตรงอีกต่อไป
    โค้ดเดิมอาจต้องแก้ไขเพื่อแปลงจาก `f64` อย่างชัดเจน
-   เทรต `Float` แบบกำหนดเองใน `rand_distr` ถูกแทนที่ด้วย
    `num_traits::Float` การ implement `Float` ให้กับชนิดข้อมูลที่ผู้ใช้กำหนด
    ต้องย้ายมาด้วย ด้วยฟังก์ชันคณิตศาสตร์จาก `num_traits::Float`
    `rand_distr` จึงรองรับ `no_std` แล้ว

นอกจากนี้ยังมีการปรับปรุงเล็กๆ อีกบางอย่าง:

-   การจัดการข้อผิดพลาดในการปัดเศษและ NaN ได้รับการปรับปรุงสำหรับ
    การแจกแจง [`WeightedIndex`]
-   การแจกแจง [`rand_distr::Exp`] ตอนนี้รองรับการกำหนดพารามิเตอร์ `lambda = 0`


## ลำดับ

ตอนนี้รองรับการสุ่มแบบถ่วงน้ำหนักโดยไม่ใส่คืนแล้ว ดู
[`rand::seq::index::sample_weighted`] และ
[`SliceRandom::choose_multiple_weighted`]

มีการเปลี่ยนแปลงที่ทำให้[ค่าผลลัพธ์เปลี่ยนไป](https://github.com/rust-random/rand/pull/1059)
สำหรับ [`IteratorRandom::choose`] ซึ่งปรับปรุงความแม่นยำและประสิทธิภาพ ยิ่งไปกว่านั้น
[`IteratorRandom::choose_stable`] ถูกเพิ่มเข้ามาเพื่อให้ทางเลือกที่แลกประสิทธิภาพกับการไม่ขึ้นกับ size hint ของอิเทอเรเตอร์

## ฟีเจอร์แฟล็ก

`StdRng` ตอนนี้ถูกควบคุมด้วยฟีเจอร์แฟล็กใหม่ชื่อ `std_rng` ซึ่งเปิดใช้เป็นค่าเริ่มต้น

ฟีเจอร์ `nightly` ไม่ได้เปิดใช้ฟีเจอร์ `simd_support` ตามอัตโนมัติอีกต่อไป หากคุณเคยพึ่งพาสิ่งนี้สำหรับการรองรับ SIMD คุณจะต้องใช้ฟีเจอร์ `simd_support` โดยตรง

## การทดสอบ

เทสต์ความเสถียรของค่าถูกเพิ่มให้กับการแจกแจงทั้งหมด ([rand#786]) ซึ่งช่วยบังคับใช้กฎของเราเกี่ยวกับการเปลี่ยนแปลงที่ทำให้ค่าผลลัพธ์เปลี่ยนไป (ดูหัวข้อ [Reproducibility])


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
