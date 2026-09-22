# การซีด RNG

ดังที่เราได้เห็นกันไปแล้ว เอาต์พุตของตัวสร้างเลขสุ่มเทียม (PRNG) จะถูกกำหนดขึ้นจากสถานะเริ่มต้นของมันเสมอ

ข้อกำหนดของขั้นตอนวิธี PRNG แต่ละตัวจะระบุวิธีสร้างสถานะเริ่มต้นจากคีย์ที่ป้อนเข้ามา โดยทั่วไปหากเป็นตัวสร้างเชิงรหัสลับ คีย์จะถูกกำหนดเป็นลำดับของไบต์ (byte sequence) หรือหากเป็น PRNG ขนาดเล็กก็มักจะใช้เพียงหนึ่งเวิร์ด (word) ซึ่งเราได้กำหนดรูปแบบมาตรฐานนี้ให้กับตัวสร้างทุกตัวผ่านเทรต [`SeedableRng`]

หมายเหตุ: การซีดไม่ได้หมายความว่าผลลัพธ์จะทำซ้ำได้ สำหรับสิ่งนั้นคุณต้องใช้ RNG ที่มีชื่อระบุและอัลกอริทึมคงที่ (เช่น `ChaCha12Rng` ไม่ใช่ `StdRng`) ดูเพิ่มเติมที่ [ความสามารถในการทำซ้ำ](https://rust-random.github.io/book/crate-reprod.html)

## ชนิดข้อมูล Seed

เรากำหนดให้ RNG ที่สามารถซีดได้ทุกตัว ต้องระบุชนิดข้อมูล [`Seed`] ที่สอดคล้องกับข้อกำหนดเทรตบาวด์
`AsMut<[u8]> + Default + Sized` (โดยทั่วไปมักเป็น `[u8; N]` ที่มีขนาด `N` คงที่) เราแนะนำให้ใช้ `[u8; 12]` หรือใหญ่กว่าสำหรับ PRNG ที่ไม่ใช่เชิงรหัสลับ และ `[u8; 32]` สำหรับ PRNG เชิงรหัสลับ

คุณสามารถกำหนดค่าซีดให้กับ PRNG ได้โดยตรงจากค่าดังกล่าวผ่านเมธอด [`SeedableRng::from_seed`]

## การซีดจาก...

### เอนโทรปีใหม่

การใช้ซีดใหม่ทำได้ง่ายด้วย [`rand::make_rng()`]:

```rust,editable
use rand::prelude::*;
use rand::rngs::ChaCha20Rng;

fn main() {
    let mut rng: ChaCha20Rng = rand::make_rng();
    println!("{}", rng.random_range(0..100));
}
```

โปรดทราบว่าวิธีนี้จำเป็นต้องเปิดใช้งานฟีเจอร์ `getrandom` ในเครต `rand_core` ด้วย

### RNG อีกตัว

แน่นอนว่าเราสามารถนำเอาต์พุตจาก RNG อีกตัวมาใช้เป็นค่าซีดได้ โดยเครตมีเมธอดอำนวยความสะดวกเตรียมไว้ให้ดังนี้:

```rust,editable
use rand::prelude::*;

fn main() {
    let mut rng = SmallRng::from_rng(&mut rand::rng());
    println!("{}", rng.random_range(0..100));
}
```

แต่หากคุณต้องการบันทึกค่าคีย์หรือซีดนั้นเก็บไว้เพื่อนำมาใช้ซ้ำในภายหลัง คุณจะต้องเขียนโค้ดระบุรายละเอียดให้ชัดเจนขึ้นอีกเล็กน้อย:

```rust,editable
use rand::prelude::*;
use rand::rngs::ChaCha8Rng;

fn main() {
    let mut seed: <ChaCha8Rng as SeedableRng>::Seed = Default::default();
    rand::rng().fill(&mut seed);
    let mut rng = ChaCha8Rng::from_seed(seed);
    println!("{}", rng.random_range(0..100));
}
```

**คำเตือนสำคัญ**: PRNG แบบเรียบง่ายบางตัว โดยเฉพาะ [`XorShiftRng`] จะทำงานแย่เมื่อถูกซีดจากตัวสร้างชนิดเดียวกัน (ในกรณีนี้ Xorshift สร้างสำเนาของตัวเอง) สำหรับ PRNG เชิงรหัสลับนี่ไม่ใช่ปัญหา สำหรับตัวอื่นๆ ขอแนะนำให้ซีดจากตัวสร้างต่างชนิดกัน โดย [`ChaCha8Rng`] ถือเป็นตัวเลือกที่ยอดเยี่ยมอย่างยิ่งในการนำมาทำเป็นตัวสร้างหลักแบบดีเทอร์มินิสติก (master generator) (แต่หากเป็นการใช้งานเชิงรหัสลับ ขอแนะนำให้เลือกใช้รุ่น 12 รอบขึ้นไป)

### ตัวเลขง่ายๆ

สำหรับบางแอปพลิเคชัน โดยเฉพาะงานจำลองทางวิทยาศาสตร์ สิ่งที่คุณต้องการอาจเป็นเพียงลำดับของค่าซีดตัวเลขคงที่ที่ไม่ซ้ำกัน เช่น 1, 2, 3 เป็นต้น

เมธอด [`SeedableRng::seed_from_u64`] ได้รับการออกแบบมาเพื่อตอบโจทย์นี้โดยเฉพาะ การทำงานภายในจะใช้ PRNG แบบง่ายในการกระจายบิตของตัวเลขอินพุตไปยังบิตต่างๆ ของซีด พร้อมทั้งมีคุณสมบัติ bit-avalanche ที่ดี (ส่งผลให้ตัวเลขที่ใกล้เคียงกัน เช่น 0 กับ 1 ถูกแปลงเป็นค่าซีดที่แตกต่างกันอย่างสิ้นเชิง และสร้างลำดับ RNG ที่เป็นอิสระต่อกัน)

```rust,editable
use rand::prelude::*;
use rand::rngs::ChaCha8Rng;

fn main() {
    let mut rng = ChaCha8Rng::seed_from_u64(2);
    println!("{}", rng.random_range(0..100));
}
```

โปรดทราบว่าตัวเลขที่มีขนาดไม่เกิน 64 บิต **ไม่สามารถมอบความปลอดภัยได้** จึงไม่ควรนำไปใช้กับงานด้านวิทยาการรหัสลับหรือเกมการพนันเป็นอันขาด

### สตริง หรือข้อมูลที่แฮชได้ทั่วไป

สมมติว่าคุณต้องการเปิดให้ผู้ใช้งานป้อนข้อความสตริงเข้ามาเป็นค่าซีดของตัวสร้างเลขสุ่ม ในอุดมคติแล้ว ทุกส่วนของข้อความสตริงควรส่งอิทธิพลต่อผลลัพธ์ของตัวสร้าง และการเปลี่ยนแปลงข้อความเพียงเล็กน้อยก็ควรให้ลำดับตัวเลขสุ่มที่เป็นอิสระต่อกันโดยสิ้นเชิง

เราสามารถบรรลุสิ่งนี้ได้โดยการใช้ฟังก์ชันแฮช เพื่อบีบอัดข้อมูลอินพุตทั้งหมดให้กลายเป็นค่าแฮช จากนั้นจึงนำผลลัพธ์แฮชดังกล่าวไปใช้เป็นซีดของตัวสร้าง โดยเครต [`rand_seeder`] ได้รับการออกแบบมาเพื่อการนี้โดยเฉพาะ

```rust,noplayground
use rand::prelude::*;
use rand::rngs::Xoshiro256PlusPlus;
use rand_seeder::{Seeder, SipHasher};

fn main() {
    // In one line:
    let mut rng: Xoshiro256PlusPlus = Seeder::from("stripy zebra").into_rng();
    println!("{}", rng.random::<char>());

    // If we want to be more explicit, first we create a SipRng:
    let hasher = SipHasher::from("a sailboat");
    let mut hasher_rng = hasher.into_rng();
    // (Note: hasher_rng is a full RNG and can be used directly.)

    // Now, we use hasher_rng to create a seed:
    let mut seed: <Xoshiro256PlusPlus as SeedableRng>::Seed = Default::default();
    hasher_rng.fill(&mut seed);

    // And create our RNG from that seed:
    let mut rng = Xoshiro256PlusPlus::from_seed(seed);
    println!("{}", rng.random::<char>());
}
```

ข้อควรระวัง: เครต `rand_seeder` **ไม่เหมาะสำหรับการใช้งานเชิงรหัสลับ** และมัน **ไม่ใช่เครื่องมือสำหรับแฮชรหัสผ่าน (password hasher)** สำหรับงานด้านการจัดการรหัสผ่าน จำเป็นต้องใช้ฟังก์ชันสร้างคีย์เฉพาะทาง (key-derivation function) เช่น Argon2 เท่านั้น


[`SeedableRng`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html
[`Seed`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#type.Seed
[`SeedableRng::from_seed`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#tymethod.from_seed
[`SeedableRng::from_rng`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#method.from_rng
[`SeedableRng::seed_from_u64`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#method.seed_from_u64
[`XorShiftRng`]: https://docs.rs/rand_xorshift/latest/rand_xorshift/struct.XorShiftRng.html
[`ChaCha8Rng`]: https://docs.rs/rand/latest/rand/rngs/struct.ChaCha8Rng.html
[`rand_seeder`]: https://github.com/rust-random/seeder/
[`rand::make_rng()`]: https://docs.rs/rand/latest/rand/fn.make_rng.html
