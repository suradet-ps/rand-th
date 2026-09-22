# ค่าสุ่ม

ตอนนี้เรามีวิธีสร้างข้อมูลสุ่มแล้ว จะแปลงมันเป็นชนิดของค่าที่เราต้องการได้อย่างไร?

นี่เป็นคำถามหลอกๆ เพราะเราจำเป็นต้องรู้ทั้ง *ช่วง* ที่ต้องการและชนิดของ *การแจกแจง* ของค่านี้ (ซึ่งเป็นเรื่องของส่วน[ถัดไป](guide-dist.md))

## เทรต `Rng`

เพื่อความสะดวก ตัวสร้างทุกตัวจะ implement เทรต [`Rng`] โดยอัตโนมัติ ซึ่งให้ทางลัดหลายวิธีสำหรับสร้างค่า เทรตนี้มีฟังก์ชันอำนวยความสะดวกสำหรับสร้างค่าที่แจกแจงแบบสม่ำเสมอ:

-   [`Rng::random`] สร้างค่าสุ่มที่ไม่เอนเอียง (สม่ำเสมอ) จากช่วงที่เหมาะสมกับ
    ชนิดข้อมูลนั้น สำหรับจำนวนเต็ม โดยปกติคือช่วงที่แทนค่าได้ทั้งหมด
    (เช่น จาก `0u32` ถึง `std::u32::MAX`) สำหรับทศนิยมคือระหว่าง 0 กับ 1
    และยังรองรับชนิดข้อมูลอื่นๆ รวมถึงแอเรย์และทูเพิล
    
    เมธอดนี้เป็นตัวห่อหุ้มที่สะดวกของการแจกแจง [`StandardUniform`]
    ตามที่อธิบายใน[ส่วนถัดไป](guide-dist.html#การแจกแจงแบบสมำเสมอ)
-   [`Rng::random_range`] สร้างค่าสุ่มที่ไม่เอนเอียงภายในช่วงที่กำหนด
-   [`Rng::fill`] และ [`Rng::try_fill`] เป็นฟังก์ชันที่ปรับแต่งประสิทธิภาพแล้วสำหรับเติมค่าสุ่มลงในสไลซ์ไบต์หรือสไลซ์จำนวนเต็มใดๆ

นอกจากนี้ยังมีฟังก์ชันอำนวยความสะดวกสำหรับสร้างค่าบูลีนแบบไม่สม่ำเสมอ:

-   [`Rng::random_bool`] สร้างค่าบูลีนด้วยความน่าจะเป็นที่กำหนด
-   [`Rng::random_ratio`] สร้างค่าบูลีนเช่นกัน โดยความน่าจะเป็นนิยามผ่านเศษส่วน

สุดท้าย มีฟังก์ชันสำหรับสุ่มจากการแจกแจงใดก็ได้:

-   [`Rng::sample`] สุ่มโดยตรงจาก[การแจกแจง](guide-dist.md)บางชนิด

ตัวอย่าง:

```rust
use rand::RngExt;
# fn main() {
let mut rng = rand::rng();

// an unbiased integer over the entire range:
let i: i32 = rng.random();
println!("i = {i}");

// a uniformly distributed value between 0 and 1:
let x: f64 = rng.random();
println!("x = {x}");

// simulate rolling a die:
println!("roll = {}", rng.random_range(1..=6));
# }
```

นอกจากนี้ ฟังก์ชัน [`random`] ยังเป็นทางลัดไปยัง [`Rng::random`] บน [`rng()`]:
```rust
# use rand::Rng;
# fn main() {
println!("Tossing a coin...");
if rand::random() {
    println!("We got lucky!");
}
# }
```

## ชนิดข้อมูลสุ่มที่กำหนดเอง

จากตัวอย่างข้างต้นจะเห็นว่า `rng.random()` ให้การแจกแจงค่าที่แตกต่างกันไปตามชนิดข้อมูล:

-   ค่า `i32` ถูกสุ่มจาก `i32::MIN ..= i32::MAX` แบบสม่ำเสมอ
-   ค่า `f32` ถูกสุ่มจาก `0.0 .. 1.0` แบบสม่ำเสมอ

นี่คือการแจกแจง [`StandardUniform`] [`Distribution`] เป็นหัวข้อของบทถัดไป แต่เนื่องจากความสำคัญของการแจกแจง [`StandardUniform`] เราจึงขอแนะนำที่นี่ ตามปกติแล้ว มาตรฐานเป็นเรื่องที่กำหนดขึ้นตามอำเภอใจ แต่ถูกเลือกตามตรรกะที่สมเหตุสมผล:

-   ค่าถูกสุ่มแบบสม่ำเสมอ: เมื่อให้ช่วงย่อยสองช่วงที่มีขนาดเท่ากัน แต่ละช่วงมีโอกาสเท่ากันที่จะมีค่าสุ่มถัดไป
-   โดยปกติใช้ช่วงทั้งหมดของชนิดข้อมูลเป้าหมาย
-   สำหรับ `f32` และ `f64` ใช้ช่วง `0.0 .. 1.0` (ไม่รวม `1.0`) ด้วยเหตุผลสองประการ: (ก) นี่เป็นแนวปฏิบัติทั่วไปของตัวสร้างเลขสุ่ม และ (ข) เพราะสำหรับหลายจุดประสงค์ การมีการแจกแจงตัวอย่างแบบสม่ำเสมอ (ตามเส้นจำนวนจริง) เป็นสิ่งสำคัญ และสิ่งนี้ทำได้เฉพาะกับการแทนค่าทศนิยมโดยการจำกัดช่วงเท่านั้น

ด้วยเหตุนี้ เราจึง implement การแจกแจง [`StandardUniform`] สำหรับชนิดข้อมูลของเราเองได้:
```rust
use rand::{Rng, RngExt};
use rand::distr::{Distribution, StandardUniform, Uniform};
use std::f64::consts::TAU; // = 2π

/// Represents an angle, in radians
#[derive(Debug)]
pub struct Angle(f64);
impl Angle {
    pub fn from_degrees(degrees: f64) -> Self {
        Angle(degrees * (std::f64::consts::TAU / 360.0))
    }
}

impl Distribution<Angle> for StandardUniform {
    fn sample<R: Rng + ?Sized>(&self, rng: &mut R) -> Angle {
        // It would be correct to write:
        // Angle(rng.random::<f64>() * TAU)

        // However, the following is preferred:
        Angle(Uniform::new(0.0, TAU).unwrap().sample(rng))
    }
}

fn main() {
    let angle: Angle = rand::rng().random();
    println!("Random angle: {angle:?}");
}
```

[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`Rng::random`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random
[`Rng::random_range`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_range
[`Rng::sample`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.sample
[`Rng::random_bool`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_bool
[`Rng::random_ratio`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_ratio
[`Rng::fill`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.fill
[`Rng::try_fill`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.try_fill
[`random`]: https://docs.rs/rand/latest/rand/fn.random.html
[`rng()`]: https://docs.rs/rand/latest/rand/fn.rng.html
[`Distribution`]: https://docs.rs/rand/latest/rand/distr/trait.Distribution.html
[`StandardUniform`]: https://docs.rs/rand/latest/rand/distr/struct.StandardUniform.html
