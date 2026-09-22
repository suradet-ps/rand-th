# RNG แบบขนาน

## ทฤษฎี: RNG หลายตัว

หากคุณต้องการใช้ตัวสร้างแบบสุ่มในเธรดทำงานหลายตัวพร้อมกัน คุณจะต้องใช้ RNG หลายตัว วิธีที่แนะนำมีดังนี้:

1.  ใช้ [`rng()`] ในแต่ละเธรดทำงาน ตัวสร้างนี้จะถูกซีดโดยอัตโนมัติ
    (แบบประเมินเมื่อต้องการ/lazy และได้ค่าซีดที่ไม่ซ้ำกัน) ในแต่ละเธรดที่ใช้งาน
2.  ใช้ [`rng()`] (หรือ RNG หลักตัวอื่น) เพื่อซีด RNG ที่กำหนดเองในแต่ละ
    เธรดทำงาน ข้อดีหลักคือความยืดหยุ่นในการเลือก RNG ที่ใช้
3.  ใช้ RNG ที่กำหนดเองต่อ *ชิ้นงานย่อย (work unit)* ไม่ใช่ต่อ *เธรดทำงาน (worker thread)* หาก RNG เหล่านี้
    ถูกซีดอย่างดีเทอร์มินิสติก ก็จะได้ผลลัพธ์ที่กำหนดได้เช่นกัน น่าเสียดายที่
    การซีด RNG ใหม่สำหรับแต่ละชิ้นงานย่อยจากตัวสร้างหลักไม่สามารถทำแบบขนานได้ จึงอาจช้า
4.  ใช้ซีดหลักเพียงค่าเดียว สำหรับแต่ละชิ้นงานย่อย ให้ซีด RNG ด้วยซีดหลัก
    แล้วตั้งสตรีมของ RNG เป็นหมายเลขชิ้นงานย่อย วิธีนี้อาจเร็วกว่า (3)
    และยังคงกำหนดผลลัพธ์ได้ แต่ไม่ใช่ RNG ทุกตัวรองรับ

หมายเหตุ: อย่าแค่โคลน RNG ให้เธรดทำงานหรือชิ้นงานย่อย สำเนาจะให้ลำดับเอาต์พุตเดิมเหมือนต้นฉบับ อย่างไรก็ตาม คุณใช้สำเนาได้หากตั้งสตรีมที่ไม่ซ้ำกันให้แต่ละตัว

### สายธาร

ตระกูล RNG ใดรองรับหลายสตรีม?

-   [ChaCha](https://docs.rs/rand/latest/rand/rngs/struct.ChaCha20Rng.html): RNG ตระกูล ChaCha
    รองรับซีด 256 บิต สตรีม 64 บิต และตัวนับ 64 บิต (ต่อบล็อก 16 คำ)
    จึงรองรับ 2<sup>64</sup> สตรีม สตรีมละ 2<sup>68</sup> คำ
-   [Hc128](https://docs.rs/rand_hc/latest/rand_hc/) เป็น RNG เชิงรหัสลับที่
    รองรับซีด 256 บิต; เราอาจสร้างซีดนี้จาก (เช่น) คีย์ 192 บิตที่เล็กกว่า
    บวกกับสตรีม 64 บิตได้

โปรดทราบว่าวิธีสร้างซีดจากคีย์ที่เล็กกว่าบวกกับตัวนับสตรีมข้างต้น แนะนำได้เฉพาะกับ PRNG เชิงรหัสลับเท่านั้น เนื่องจาก RNG แบบง่ายมักมีสหสัมพันธ์ในเอาต์พุตเมื่อใช้คีย์สองค่าที่คล้ายกัน และอาจต้องใช้ซีดที่ "ดูสุ่ม" เพื่อให้ได้เอาต์พุตคุณภาพสูง

PRNG ที่ไม่ใช่เชิงรหัสลับอาจยังรองรับหลายสตรีมได้ แต่น่าจะมีข้อจำกัดที่สำคัญ (โดยเฉพาะอย่างยิ่ง เมื่อคำแนะนำทั่วไปของ PRNG เหล่านี้คือไม่ควรใช้เกินรากที่สองของคาบของตัวสร้าง)

-   [Xoshiro](https://docs.rs/rand_xoshiro/latest/rand_xoshiro/): RNG ตระกูล Xoshiro
    รองรับเมธอด `jump` และ `long_jump` ซึ่งใช้แบ่งเอาต์พุตของ RNG เดียว
    ออกเป็นหลายสตรีมได้อย่างมีประสิทธิผล ในทางปฏิบัติสิ่งนี้มีประโยชน์เฉพาะ
    เมื่อมีสตรีมจำนวนน้อย เพราะต้องเรียก `jump` จำนวน `n` ครั้งเพื่อเลือกสตรีมที่ n
-   [Pcg](https://docs.rs/rand_pcg/latest/rand_pcg/): RNG เหล่านี้รองรับ
    การสร้างด้วยพารามิเตอร์ `state` และ `stream` อย่างไรก็ตาม โปรดทราบว่า
    RNG เหล่านี้ถูกวิจารณ์ว่าหลายสตรีมที่ใช้คีย์เดียวกันมักมีสหสัมพันธ์กันอย่างมาก
    ดู[ความเห็นของผู้เขียน PCG เอง](https://www.pcg-random.org/posts/critiquing-pcg-streams.html)

    RNG ตระกูล PCG *ยัง*รองรับเมธอด `fn advance(delta)` ซึ่งอาจใช้แบ่ง
    สตรีมเดียวออกเป็นหลายสตรีมย่อยได้เช่นเดียวกับ `jump` ของ Xoshiro
    (แต่ดีกว่าเพราะระบุออฟเซตได้)

## ภาคปฏิบัติ: แบบหลายเธรดที่ไม่กำหนดได้

เราใช้[อิเทอเรเตอร์แบบขนาน (parallel iterators)](https://docs.rs/rayon/latest/rayon/iter/index.html) ของ Rayon โดยใช้ [`map_init`] เพื่อเริ่มต้น RNG ในแต่ละเธรดทำงาน หมายเหตุ: RNG นี้อาจถูกใช้ซ้ำข้ามหลายชิ้นงานย่อย ซึ่งอาจถูกแบ่งระหว่างเธรดทำงานแบบไม่กำหนดได้

```rust
use rand::distr::{Distribution, Uniform};
use rayon::prelude::*;

static SAMPLES: u64 = 1_000_000;

fn main() {
    let range = Uniform::new(-1.0f64, 1.0).unwrap();

    let in_circle = (0..SAMPLES)
        .into_par_iter()
        .map_init(|| rand::rng(), |rng, _| {
            let a = range.sample(rng);
            let b = range.sample(rng);
            if a * a + b * b <= 1.0 {
                1
            } else {
                0
            }
        })
        .reduce(|| 0usize, |a, b| a + b);

    // prints something close to 3.14159...
    println!(
        "π is approximately {}",
        4. * (in_circle as f64) / (SAMPLES as f64)
    );
}
```

## ภาคปฏิบัติ: แบบหลายเธรดที่กำหนดได้

เราใช้วิธี (4) ข้างต้นเพื่อให้ได้ผลลัพธ์ที่กำหนดได้: เริ่มต้น RNG ทั้งหมดจากซีดเดียว แต่ใช้หลายสตรีม
เราใช้ [`ChaCha8Rng::set_stream`] เพื่อทำเช่นนี้

โปรดทราบเพิ่มเติมว่าเราจัดกลุ่มชิ้นงานย่อยหลายหน่วยด้วยกันตาม `BATCH_SIZE` ด้วยมือ สิ่งนี้สำคัญเพราะค่าใช้จ่ายในการเริ่มต้น RNG สูงเมื่อเทียบกับค่าใช้จ่ายของชิ้นงานย่อยเรา (สร้างค่าสุ่มสองค่าบวกการคำนวณเล็กน้อย) การจัดกลุ่มด้วยมืออาจช่วยเพิ่มประสิทธิภาพของการจำลองแบบไม่กำหนดได้ข้างต้นด้วย

(หมายเหตุ: ตัวอย่างนี้อยู่ที่ <https://github.com/rust-random/rand/blob/master/examples/rayon-monte-carlo.rs>)

```rust
use rand::distr::{Distribution, Uniform};
use rand::{SeedableRng, rngs::ChaCha8Rng};
use rayon::prelude::*;

static SEED: u64 = 0;
static BATCH_SIZE: u64 = 10_000;
static BATCHES: u64 = 1000;

fn main() {
    let range = Uniform::new(-1.0f64, 1.0).unwrap();

    let in_circle = (0..BATCHES)
        .into_par_iter()
        .map(|i| {
            let mut rng = ChaCha8Rng::seed_from_u64(SEED);
            rng.set_stream(i);
            let mut count = 0;
            for _ in 0..BATCH_SIZE {
                let a = range.sample(&mut rng);
                let b = range.sample(&mut rng);
                if a * a + b * b <= 1.0 {
                    count += 1;
                }
            }
            count
        })
        .reduce(|| 0usize, |a, b| a + b);

    // prints 3.1409052 (deterministic and reproducible result)
    println!(
        "π is approximately {}",
        4. * (in_circle as f64) / ((BATCH_SIZE * BATCHES) as f64)
    );
}
```

[`rng()`]: https://docs.rs/rand/latest/rand/fn.rng.html
[`map_init`]: https://docs.rs/rayon/latest/rayon/iter/trait.ParallelIterator.html#method.map_init
[`ChaCha8Rng::set_stream`]: https://docs.rs/rand/latest/rand/rngs/struct.ChaCha8Rng.html#method.set_stream
