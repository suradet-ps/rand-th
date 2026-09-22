# เริ่มต้นอย่างรวดเร็ว

ด้านล่างนี้คือตัวอย่างสั้นๆ หากต้องการดูเพิ่มเติม โปรดดูที่ [เอกสารอ้างอิง API]
หรือ[คู่มือ]

มาเริ่มกันด้วยตัวอย่าง

```rust,editable
// import commonly used items from the prelude:
use rand::prelude::*;

fn main() {
    // We can use random() immediately. It can produce values of many common types:
    let x: u8 = rand::random();
    println!("{}", x);

    if rand::random() { // generates a boolean
        println!("Heads!");
    }

    // If we want to be a bit more explicit (and a little more efficient) we can
    // make a handle to the thread-local generator:
    let mut rng = rand::rng();
    if rng.random() { // random bool
        let x: f64 = rng.random(); // random number in range [0, 1)
        let y = rng.random_range(-10.0..10.0);
        println!("x is: {}", x);
        println!("y is: {}", y);
    }

    println!("Dice roll: {}", rng.random_range(1..=6));
    println!("Number from 0 to 9: {}", rng.random_range(0..10));
    
    // Sometimes it's useful to use distributions directly:
    let distr = rand::distr::Uniform::new_inclusive(1, 100).unwrap();
    let mut nums = [0i32; 3];
    for x in &mut nums {
        *x = rng.sample(distr);
    }
    println!("Some numbers: {:?}", nums);

    // We can also interact with iterators and slices:
    let arrows_iter = "➡⬈⬆⬉⬅⬋⬇⬊".chars();
    println!("Lets go in this direction: {}", arrows_iter.choose(&mut rng).unwrap());
    let mut nums = [1, 2, 3, 4, 5];
    nums.shuffle(&mut rng);
    println!("I shuffled my {:?}", nums);
}
```

สิ่งแรกที่คุณอาจสังเกตเห็นคือเรา import ทุกอย่างจาก [prelude] นี่คือวิธีที่ขี้เกียจที่สุดในการ `use` rand และเช่นเดียวกับ
[prelude ของไลบรารีมาตรฐาน](https://doc.rust-lang.org/std/prelude/)
มันจะ import เฉพาะรายการที่ใช้บ่อยที่สุดเท่านั้น หากคุณไม่ต้องการใช้ prelude
อย่าลืม import เทรต [`Rng`] ด้วย!

ไลบรารี Rand จะเริ่มต้นตัวสร้างแบบเธรดโลคอลที่มีความปลอดภัยให้โดยอัตโนมัติเมื่อมีการใช้งาน
ซึ่งเข้าถึงได้ผ่านฟังก์ชัน [`rng()`] และ [`random`] สำหรับรายละเอียดเพิ่มเติมในหัวข้อนี้ ดูที่ [ตัวสร้างแบบสุ่ม](guide-gen.md)

ในขณะที่ฟังก์ชัน [`random`] สุ่มได้เฉพาะค่าที่อยู่ใน [`StandardUniform`]
(ซึ่งขึ้นอยู่กับชนิดข้อมูล) แต่ [`rng()`] จะคืนตัวจัดการ (handle) ของตัวสร้างให้คุณ
ตัวสร้างทุกตัว implement เทรต [`Rng`] ซึ่งมีเมธอด [`random`],
[`random_range`] และ [`sample`] ที่ใช้ในตัวอย่างข้างต้น

Rand ยังมีฟังก์ชันสำหรับอิเทอเรเตอร์และสไลซ์ผ่านอีกสองเทรต คือ
[`IteratorRandom`] และ [`SliceRandom`]

## RNG ที่ใช้ซีดแบบคงที่

คุณอาจสังเกตเห็นการใช้ `rand::rng()` ข้างต้นและสงสัยว่าจะกำหนดซีดแบบคงที่ได้อย่างไร
ในการทำเช่นนั้น คุณต้องระบุ RNG แล้วใช้เมธอดอย่าง [`seed_from_u64`] หรือ [`from_seed`]

โปรดทราบว่า [`seed_from_u64`] **ไม่เหมาะสำหรับการใช้งานเชิงรหัสลับ**
เนื่องจาก `u64` เพียงค่าเดียวไม่สามารถให้เอนโทรปีที่เพียงพอต่อการซีด RNG ได้อย่างปลอดภัย
RNG เชิงรหัสลับทุกตัวรับซีดที่เหมาะสมกว่าผ่าน [`from_seed`]

เราใช้ `ChaCha8Rng` ด้านล่างเพราะมันเร็ว พอร์ตได้ และมีคุณภาพดี
ดู RNG อื่นๆ ได้ที่ส่วน [RNGs] แต่หลีกเลี่ยง `SmallRng` และ `StdRng`
หากคุณให้ความสำคัญกับผลลัพธ์ที่ทำซ้ำได้

```rust,editable
use rand::{rngs::ChaCha8Rng, RngExt, SeedableRng};

fn main() {
    let mut rng = ChaCha8Rng::seed_from_u64(10);
    println!("Random f32: {}", rng.random::<f32>());
}
```

[เอกสารอ้างอิง API]: https://docs.rs/rand/
[คู่มือ]: guide.md
[RNGs]: guide-rngs.md
[prelude]: https://docs.rs/rand/latest/rand/prelude/
[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`random`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random
[`random_range`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_range
[`sample`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.sample
[`rng()`]: https://docs.rs/rand/latest/rand/fn.rng.html
[`random`]: https://docs.rs/rand/latest/rand/fn.random.html
[`StandardUniform`]: https://docs.rs/rand/latest/rand/distr/struct.StandardUniform.html
[`IteratorRandom`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html
[`SliceRandom`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html
[`seed_from_u64`]: https://docs.rs/rand/latest/rand/trait.SeedableRng.html#method.seed_from_u64
[`from_seed`]: https://docs.rs/rand/latest/rand/trait.SeedableRng.html#tymethod.from_seed
