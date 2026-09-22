# การซีด RNG

ดังที่เราได้เห็นแล้ว เอาต์พุตของตัวสร้างเลขสุ่มเทียม (PRNG) ถูกกำหนดโดยสถานะเริ่มต้นของมัน

นิยามของ PRNG บางตัวระบุว่าสถานะเริ่มต้นควรสร้างจากคีย์อย่างไร โดยปกติคีย์จะระบุเป็นลำดับไบต์สำหรับตัวสร้างเชิงรหัสลับ หรือสำหรับ PRNG ขนาดเล็กมักเป็นเพียงคำหนึ่งคำ เราทำให้เรื่องนี้เป็นทางการสำหรับตัวสร้างทุกตัวของเราด้วยเทรต [`SeedableRng`]

หมายเหตุ: การซีดไม่ได้หมายความว่าผลลัพธ์จะทำซ้ำได้ สำหรับสิ่งนั้นคุณต้องใช้ RNG ที่มีชื่อระบุและอัลกอริทึมคงที่ (เช่น `ChaCha12Rng` ไม่ใช่ `StdRng`) ดูเพิ่มเติมที่ [ความสามารถในการทำซ้ำ](https://rust-random.github.io/book/crate-reprod.html)

## ชนิดข้อมูล Seed

เรากำหนดให้ RNG ที่ซีดได้ทุกตัวนิยามชนิดข้อมูล [`Seed`] ที่สอดคล้องกับ
`AsMut<[u8]> + Default + Sized` (โดยปกติคือ `[u8; N]` สำหรับ `N` คงที่) เราแนะนำให้ใช้ `[u8; 12]` หรือใหญ่กว่าสำหรับ PRNG ที่ไม่ใช่เชิงรหัสลับ และ `[u8; 32]` สำหรับ PRNG เชิงรหัสลับ

PRNG อาจถูกซีดโดยตรงจากค่าดังกล่าวด้วย [`SeedableRng::from_seed`]

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

โปรดทราบว่าสิ่งนี้ต้องให้ `rand_core` เปิดใช้ฟีเจอร์ `getrandom`

### RNG อีกตัว

เห็นได้ชัดว่า RNG อีกตัวอาจถูกใช้เพื่อเติมซีดได้ เรามีเมธอดอำนวยความสะดวกสำหรับเรื่องนี้:

```rust,editable
use rand::prelude::*;

fn main() {
    let mut rng = SmallRng::from_rng(&mut rand::rng());
    println!("{}", rng.random_range(0..100));
}
```

แต่สมมติว่าคุณต้องการบันทึกคีย์ไว้ใช้ภายหลัง สำหรับสิ่งนั้นคุณต้องเขียนให้ชัดเจนขึ้นเล็กน้อย:

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

**คำเตือนที่ต้องกล่าวไว้**: PRNG แบบง่ายบางตัว โดยเฉพาะ [`XorShiftRng`] จะทำงานแย่เมื่อถูกซีดจากตัวสร้างชนิดเดียวกัน (ในกรณีนี้ Xorshift สร้างสำเนาของตัวเอง) สำหรับ PRNG เชิงรหัสลับนี่ไม่ใช่ปัญหา สำหรับตัวอื่นๆ แนะนำให้ซีดจากตัวสร้างคนละชนิด [`ChaCha8Rng`] เป็นตัวเลือกที่ยอดเยี่ยมสำหรับเป็นตัวสร้างหลักแบบดีเทอร์มินิสติก (แต่สำหรับการใช้งานเชิงรหัสลับ ให้เลือกแบบ 12 รอบหรือมากกว่า)

### ตัวเลขง่ายๆ

สำหรับบางแอปพลิเคชัน โดยเฉพาะการจำลอง สิ่งที่คุณต้องการคือลำดับของซีดเลขสุ่มแบบคงที่ที่แตกต่างกัน เช่น 1, 2, 3 เป็นต้น

[`SeedableRng::seed_from_u64`] ถูกออกแบบมาเพื่อกรณีนี้โดยเฉพาะ ภายในการทำงาน มันใช้ PRNG แบบง่ายเพื่อเติมบิตของซีดจากตัวเลขอินพุต พร้อมทั้งให้การกระจายบิต (bit-avalanche) ที่ดี (เพื่อให้ตัวเลขที่คล้ายกัน เช่น 0 กับ 1 แปลงเป็นซีดที่แตกต่างกันมากและได้ลำดับ RNG ที่เป็นอิสระต่อกัน)

```rust,editable
use rand::prelude::*;
use rand::rngs::ChaCha8Rng;

fn main() {
    let mut rng = ChaCha8Rng::seed_from_u64(2);
    println!("{}", rng.random_range(0..100));
}
```

โปรดทราบว่าตัวเลข 64 บิตหรือน้อยกว่า **ไม่สามารถปลอดภัยได้** ดังนั้นจึงไม่ควรใช้กับแอปพลิเคชันอย่างการเข้ารหัสหรือเกมพนัน

### สตริง หรือข้อมูลที่แฮชได้ทั่วไป

สมมติว่าคุณให้ผู้ใช้ป้อนสตริงเพื่อซีดตัวสร้างเลขสุ่ม ในอุดมคติแล้ว ทุกส่วนของสตริงควรมีผลต่อตัวสร้าง และการเปลี่ยนแปลงสตริงเพียงเล็กน้อยควรให้ลำดับตัวสร้างที่เป็นอิสระต่อกันโดยสมบูรณ์

สิ่งนี้ทำได้โดยใช้ฟังก์ชันแฮชเพื่อบีบอัดข้อมูลอินพุตทั้งหมดให้เป็นผลลัพธ์แฮช แล้วใช้ผลลัพธ์นั้นซีดตัวสร้าง เครต [`rand_seeder`] ถูกออกแบบมาเพื่อจุดประสงค์นี้โดยเฉพาะ

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

โปรดทราบว่า `rand_seeder` **ไม่เหมาะ**สำหรับการใช้งานเชิงรหัสลับ มัน**ไม่ใช่ตัวแฮชรหัสผ่าน** สำหรับการใช้งานดังกล่าวต้องใช้ฟังก์ชันสืบทอดคีย์ (key-derivation function) เช่น Argon2


[`SeedableRng`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html
[`Seed`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#type.Seed
[`SeedableRng::from_seed`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#tymethod.from_seed
[`SeedableRng::from_rng`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#method.from_rng
[`SeedableRng::seed_from_u64`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html#method.seed_from_u64
[`XorShiftRng`]: https://docs.rs/rand_xorshift/latest/rand_xorshift/struct.XorShiftRng.html
[`ChaCha8Rng`]: https://docs.rs/rand/latest/rand/rngs/struct.ChaCha8Rng.html
[`rand_seeder`]: https://github.com/rust-random/seeder/
[`rand::make_rng()`]: https://docs.rs/rand/latest/rand/fn.make_rng.html
