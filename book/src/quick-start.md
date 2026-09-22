# เริ่มต้นอย่างรวดเร็ว

ด้านล่างนี้คือตัวอย่างการใช้งานสั้นๆ เพื่อให้เห็นภาพรวม หากต้องการศึกษาเพิ่มเติม โปรดดูที่ [เอกสารอ้างอิง API]
หรือ[คู่มือ]

มาเริ่มต้นด้วยตัวอย่างโค้ดกันเลย:

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

สิ่งแรกที่คุณอาจสังเกตเห็นคือเราได้ import ทุกอย่างมาจาก [prelude] ซึ่งถือเป็นวิธีทางลัดที่สะดวกที่สุดในการเรียก `use` ฟังก์ชันต่างๆ ของ rand เช่นเดียวกับ
[prelude ของไลบรารีมาตรฐาน](https://doc.rust-lang.org/std/prelude/)
ที่จะนำเข้าเฉพาะไอเท็มที่ใช้งานบ่อยที่สุดเท่านั้น หากคุณไม่ต้องการใช้ prelude
ก็อย่าลืม import เทรต [`Rng`] เข้ามาด้วย!

ไลบรารี Rand จะเริ่มต้นสร้างตัวสร้างเลขสุ่มระดับเธรด (thread-local generator) ที่มีความปลอดภัยให้โดยอัตโนมัติเมื่อมีการเรียกใช้งาน
ซึ่งสามารถเข้าถึงได้ผ่านฟังก์ชัน [`rng()`] และ [`random`] สำหรับรายละเอียดเพิ่มเติมในหัวข้อนี้ โปรดดูที่ [ตัวสร้างแบบสุ่ม](guide-gen.md)

ในขณะที่ฟังก์ชัน [`random`] สามารถสุ่มได้เฉพาะค่าตามการแจกแจง [`StandardUniform`]
(ซึ่งขึ้นอยู่กับชนิดข้อมูลที่กำหนด) แต่การเรียก [`rng()`] จะคืนค่าแฮนเดิล (handle) ของตัวสร้างมาให้คุณ
ซึ่งตัวสร้างทุกตัวจะอิมพลีเมนต์เทรต [`Rng`] ที่มีเมธอดอำนวยความสะดวกมากมาย เช่น เมธอด [`random`],
[`random_range`] และ [`sample`] ตามที่เห็นในตัวอย่างข้างต้น

นอกจากนี้ Rand ยังมอบฟังก์ชันการทำงานร่วมกับอิเทอเรเตอร์และสไลซ์ผ่านอีกสองเทรต คือ
[`IteratorRandom`] และ [`SliceRandom`]

## RNG ที่ใช้ซีดแบบคงที่

หลายท่านที่เห็นการเรียกใช้ `rand::rng()` ข้างต้น อาจสงสัยว่าจะกำหนดค่าซีด (seed) แบบคงที่ได้อย่างไร
หากต้องการทำเช่นนั้น คุณจะต้องระบุตัวสร้าง RNG ที่ต้องการ แล้วเรียกใช้เมธอดอย่าง [`seed_from_u64`] หรือ [`from_seed`]

โปรดทราบว่า [`seed_from_u64`] **ไม่เหมาะสำหรับการใช้งานเชิงรหัสลับ (cryptographic uses)**
เนื่องจากค่า `u64` เพียงค่าเดียวมีเอนโทรปีไม่เพียงพอที่จะซีด RNG ได้อย่างปลอดภัย
โดย RNG เชิงรหัสลับทุกตัวจะรองรับการรับค่าซีดที่เหมาะสมกว่าผ่านเมธอด [`from_seed`]

ในตัวอย่างด้านล่าง เราเลือกใช้ `ChaCha8Rng` เนื่องจากทำงานได้รวดเร็ว พอร์ตข้ามแพลตฟอร์มได้ และให้คุณภาพการสุ่มที่ดี
หากต้องการดูตัวสร้าง RNG ตัวอื่นๆ สามารถดูเพิ่มเติมได้ที่หัวข้อ [RNGs] แต่ขอแนะนำให้หลีกเลี่ยง `SmallRng` และ `StdRng`
หากคุณให้ความสำคัญกับผลลัพธ์ที่ทำซ้ำได้แน่นอน (reproducible)

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
