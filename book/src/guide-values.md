# ค่าสุ่ม

เมื่อเรามีเครื่องมือในการผลิตข้อมูลสุ่มดิบแล้ว คำถามถัดมาคือ เราจะแปลงข้อมูลเหล่านั้นให้กลายเป็นค่าของชนิดข้อมูลที่เราต้องการใช้งานได้อย่างไร?

นี่เป็นคำถามที่มีคำตอบสองชั้น: เพราะเราจำเป็นต้องทราบทั้ง *ขอบเขตช่วงค่า (range)* ที่ต้องการ และชนิดของ *การแจกแจง (distribution)* ของค่านั้นๆ (ซึ่งจะเป็นประเด็นหลักในบท[ถัดไป](guide-dist.md))

## เทรต `Rng`

เพื่อความสะดวกในการพัฒนา ตัวสร้างเลขสุ่มทุกตัวจะอิมพลีเมนต์เทรต [`Rng`] ให้โดยอัตโนมัติ ซึ่งทำหน้าที่เป็นทางลัดสำหรับสร้างค่าสุ่มในรูปแบบต่างๆ มากมาย โดยมีฟังก์ชันอำนวยความสะดวกสำหรับการสร้างค่าที่มีการแจกแจงแบบสม่ำเสมอ (Uniform) ดังนี้:

-   [`Rng::random`] สุ่มสร้างค่าที่ไม่ลำเอียง (แบบสม่ำเสมอ) จากช่วงค่าที่เหมาะสมตามแต่ละชนิดข้อมูล
    สำหรับจำนวนเต็มโดยทั่วไปจะครอบคลุมช่วงค่าทั้งหมดที่ชนิดข้อมูลนั้นแทนค่าได้
    (เช่น จาก `0u32` ถึง `std::u32::MAX`) สำหรับเลขทศนิยมจะอยู่ระหว่าง 0 ถึง 1
    และยังรองรับชนิดข้อมูลอื่นๆ เช่น อาร์เรย์ และ ทูเพิล
    
    เมธอดนี้เป็น wrapper อำนวยความสะดวกที่ห่อหุ้มการแจกแจง [`StandardUniform`] เอาไว้
    ตามที่อธิบายไว้ใน[ส่วนถัดไป](guide-dist.html#การแจกแจงแบบสมำเสมอ)
-   [`Rng::random_range`] สุ่มสร้างค่าที่ไม่ลำเอียงภายในช่วงค่าที่ระบุ
-   [`Rng::fill`] และ [`Rng::try_fill`] ฟังก์ชันที่ได้รับการปรับแต่งประสิทธิภาพมาเป็นพิเศษสำหรับเติมค่าสุ่มลงในสไลซ์ของไบต์หรือสไลซ์ของจำนวนเต็มใดๆ

นอกจากนี้ยังมีฟังก์ชันอำนวยความสะดวกสำหรับการสร้างค่าบูลีนแบบไม่สม่ำเสมอ:

-   [`Rng::random_bool`] สุ่มค่าบูลีนตามความน่าจะเป็นที่กำหนด
-   [`Rng::random_ratio`] สุ่มค่าบูลีนเช่นกัน โดยกำหนดความน่าจะเป็นในรูปของเศษส่วน

และสุดท้าย มีฟังก์ชันสำหรับสุ่มค่าจากการแจกแจงใดๆ ก็ได้:

-   [`Rng::sample`] สุ่มค่าโดยตรงจากการแจกแจง ([distribution](guide-dist.md)) ที่กำหนด

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

จากตัวอย่างข้างต้น จะสังเกตเห็นว่าการเรียก `rng.random()` จะให้การแจกแจงของค่าที่แตกต่างกันไปตามชนิดข้อมูล:

-   ค่า `i32` จะถูกสุ่มขึ้นจากช่วง `i32::MIN ..= i32::MAX` แบบสม่ำเสมอ
-   ค่า `f32` จะถูกสุ่มขึ้นจากช่วง `0.0 .. 1.0` แบบสม่ำเสมอ

นี่คือพฤติกรรมของการแจกแจง [`StandardUniform`] ซึ่งแม้ว่าเรื่องของ [`Distribution`] จะเป็นหัวข้อหลักในบทถัดไป แต่ด้วยความสำคัญของการแจกแจงแบบ [`StandardUniform`] เราจึงขอกล่าวถึงไว้ล่วงหน้า ณ ที่นี้ โดยตามธรรมเนียมทั่วไปแล้ว นิยามคำว่ามาตรฐานอาจดูเหมือนเป็นการกำหนดขึ้นเอง (somewhat arbitrary) แต่แท้จริงแล้วถูกคัดเลือกมาอย่างมีหลักการและมีเหตุผลรองรับเป็นอย่างดี:

-   ค่าจะถูกสุ่มแบบสม่ำเสมอ: เมื่อพิจารณาช่วงย่อยสองช่วงใดๆ ที่มีขนาดเท่ากัน แต่ละช่วงย่อมมีโอกาสเท่ากันที่จะเป็นที่อยู่ของค่าสุ่มถัดไป
-   โดยทั่วไปจะใช้ช่วงค่าทั้งหมดที่ชนิดข้อมูลเป้าหมายสามารถรองรับได้
-   สำหรับ `f32` และ `f64` จะใช้ช่วง `0.0 .. 1.0` (ไม่นับรวม `1.0`) ด้วยเหตุผลสองประการ: (ก) เป็นแนวทางปฏิบัติมาตรฐานสากลของตัวสร้างเลขสุ่ม และ (ข) ในหลายแอปพลิเคชัน การรักษาการแจกแจงตัวอย่างให้สม่ำเสมอบนเส้นจำนวนจริง (Real number line) ถือเป็นเรื่องสำคัญยิ่ง ซึ่งสำหรับระบบเลขทศนิยมแล้ว สิ่งนี้จะเป็นไปได้ก็ต่อเมื่อจำกัดช่วงค่าไว้เท่านั้น

จากหลักการดังกล่าว เราสามารถอิมพลีเมนต์การแจกแจง [`StandardUniform`] ให้กับชนิดข้อมูลที่เราสร้างขึ้นเองได้เช่นกัน:
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
