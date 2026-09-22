# การอัปเดตเป็น 0.9

เนื้อหาส่วนนี้เป็นคำแนะนำสำหรับการปรับย้ายโค้ดของคุณจาก
`rand 0.8` และ `rand_distr 0.4` ไปเป็น `rand 0.9` และ `rand_distr 0.5`

คู่มือการย้ายเวอร์ชันนี้จะเน้นไปที่การเปลี่ยนแปลงที่ส่งผลกระทบต่อความเข้ากันได้ของโค้ดเดิม (breaking changes) สำหรับรายการการเปลี่ยนแปลงทั้งหมด สามารถตรวจสอบได้จากบันทึกการเปลี่ยนแปลงที่เกี่ยวข้อง:

-   [CHANGELOG.md](https://github.com/rust-random/rand/blob/master/CHANGELOG.md)
-   [rand_core/CHANGELOG.md](https://docs.rs/crate/rand_core/latest/source/CHANGELOG.md)
-   [rand_distr/CHANGELOG.md](https://github.com/rust-random/rand/blob/master/rand_distr/CHANGELOG.md)

## ฟังก์ชันและเมธอดที่เปลี่ยนชื่อ

ใน Rust edition 2024 คำว่า [`gen` ได้กลายเป็นคีย์เวิร์ดสงวน][gen-keyword] ทำให้การเรียกใช้ไวยากรณ์ดิบอย่าง `r#gen()` ดูไม่สะดวก จึงได้มีการเปลี่ยนชื่อเมธอดบางตัวใน `rand::Rng`:
- `gen` -> `random`
- `gen_range` -> `random_range`
- `gen_bool` -> `random_bool`
- `gen_ratio` -> `random_ratio`

นอกจากนี้ `rand::thread_rng()` ได้รับการเปลี่ยนชื่อเป็น `rng()` ที่สั้นกระชับและเข้าใจง่ายขึ้น

ทั้งนี้ชื่อเดิมทั้งหมดยังคงมีอยู่ให้เรียกใช้ได้ แต่ถูกทำเครื่องหมายเลิกใช้งาน (deprecated) แล้ว

[gen-keyword]: https://doc.rust-lang.org/edition-guide/rust-2024/gen-keyword.html

## ความปลอดภัย

ใน [#1514](https://github.com/rust-random/rand/pull/1514) ได้มีข้อสรุปที่ชัดเจนว่า
"rand ไม่ใช่ไลบรารีสำหรับการเข้ารหัส" การเปลี่ยนแปลงนี้ชี้แจงให้เห็นว่า:

1.  ไลบรารี Rand เป็นโครงการพัฒนาโดยชุมชนที่ไม่มีการรับประกันผูกพันทางกฎหมายใดๆ ทั้งสิ้น
2.  ไลบรารี Rand มอบฟังก์ชันสำหรับสร้างตัวเลขสุ่มที่คาดเดาไม่ได้ แต่ไม่ได้เตรียมฟังก์ชันการทำงานด้านวิทยาการรหัสลับระดับสูงใดๆ ไว้ให้
3.  `rand::rngs::OsRng` เป็นตัวสร้างเลขสุ่มแบบไร้สถานะ จึงไม่มีสถานะภายในที่จะรั่วไหล และไม่จำเป็นต้องกำหนดซีดหรือกำหนดซีดใหม่
4.  `rand::rngs::ThreadRng` เป็นตัวสร้างเลขสุ่มที่กำหนดซีดอัตโนมัติและมีการกำหนดซีดใหม่เป็นระยะโดยใช้อัลกอริทึมสุ่มเทียมที่มีความแข็งแกร่งเชิงรหัสลับ แต่ไม่มีกลไกป้องกันสถานะในหน่วยความจำ โดยเฉพาะอย่างยิ่งไม่มีการล้างหน่วยความจำเป็นศูนย์โดยอัตโนมัติเมื่อสิ้นสุดการทำงาน ยิ่งไปกว่านั้น การออกแบบของมันเกิดจากการประนีประนอม เพื่อให้เป็น "ตัวสร้างที่เร็วและปลอดภัยอย่างสมเหตุสมผล"

นอกจากนี้ กลไกป้องกันกระบวนการ fork ที่เคยมีอยู่อย่างจำกัดสำหรับ [`ReseedingRng`] และ [`ThreadRng`] ได้ถูกถอดออกไปใน [#1379](https://github.com/rust-random/rand/pull/1379) โดยแนะนำให้การกำหนดซีดใหม่เป็นความรับผิดชอบของโค้ดส่วนที่สั่ง fork กระบวนการโดยตรง (ดูรายละเอียดเพิ่มเติมได้ในเอกสารของ [`ThreadRng`]):
```rust,ignore
fn do_fork() {
    let pid = unsafe { libc::fork() };
    if pid == 0 {
        // Reseed ThreadRng in child processes:
        rand::rng().reseed();
    }
}
```

## ดีเพนเดนซี

เครตในระบบของ Rand ต้องการ **`rustc`** เวอร์ชัน 1.63.0 ขึ้นไป

ดีเพนเดนซีของ **`getrandom`** ได้รับการปรับขึ้นเป็นเวอร์ชัน 0.3
ซึ่งใน [เวอร์ชันนี้](https://github.com/rust-random/getrandom/blob/master/CHANGELOG.md#030---2025-01-25)
มีการเปลี่ยนแปลงที่กระทบความเข้ากันได้ของบางแพลตฟอร์ม (โดยเฉพาะ WASM ที่ได้รับผลกระทบเป็นพิเศษ)

### ฟีเจอร์

ฟีเจอร์แฟล็ก:

-   `serde1` ถูกเปลี่ยนชื่อเป็น `serde`
-   `getrandom` ถูกเปลี่ยนชื่อเป็น `os_rng`
-   `thread_rng` เป็นฟีเจอร์ใหม่ (เปิดใช้เป็นค่าเริ่มต้น) ซึ่งเป็นฟีเจอร์ที่ [`rng()`] ต้องการ ([`ThreadRng`])
-   `small_rng` ตอนนี้เปิดใช้เป็นค่าเริ่มต้น
-   `rand_chacha` ไม่ใช่ฟีเจอร์ (โดยนัย) อีกต่อไป; ให้ใช้ `std_rng` แทน

## เทรตหลัก

ใน [#1424](https://github.com/rust-random/rand/pull/1424) ได้เพิ่มเทรตใหม่คือ [`TryRngCore`] เข้ามาใน [`rand_core`]:
```rust,ignore
pub trait TryRngCore {
    /// The type returned in the event of a RNG error.
    type Error: fmt::Debug + fmt::Display;

    /// Return the next random `u32`.
    fn try_next_u32(&mut self) -> Result<u32, Self::Error>;
    /// Return the next random `u64`.
    fn try_next_u64(&mut self) -> Result<u64, Self::Error>;
    /// Fill `dest` entirely with random data.
    fn try_fill_bytes(&mut self, dst: &mut [u8]) -> Result<(), Self::Error>;

    // [Provided methods hidden]
}
```
เทรตนี้เป็นแบบเจเนอริกที่รองรับทั้ง RNG ที่อาจเกิดข้อผิดพลาดได้และไม่เกิดข้อผิดพลาด (กลุ่มหลังจะใช้ประเภทข้อผิดพลาด `Error` เป็น [`Infallible`]) ในขณะที่เทรต [`RngCore`] ในปัจจุบันจะใช้เป็นตัวแทนเฉพาะ RNG ที่ไม่เกิดข้อผิดพลาดเท่านั้น

เทรต [`CryptoRng`] ตอนนี้เป็นเทรตย่อยของ [`RngCore`] มีเทรตที่เข้าคู่กันชื่อ [`TryCryptoRng`] ไว้ใช้ทำเครื่องหมายผู้ที่ implement [`TryRngCore`] ซึ่งแข็งแกร่งเชิงรหัสลับ

### การซีด RNG

เทรต [`SeedableRng`] มีการปรับเปลี่ยนบางประการ:

-   `type Seed` เพิ่ม trait bounds ได้แก่ `Clone` และ `AsRef<[u8]>`
-   `fn from_rng` เปลี่ยนชื่อเป็น `try_from_rng` พร้อมกับเพิ่มตัวแปรแบบไม่เกิดข้อผิดพลาดเป็น `from_rng` ตัวใหม่
-   `fn from_entropy` เปลี่ยนชื่อเป็น `from_os_rng` ควบคู่ไปกับตัวแปรแบบอาจเกิดข้อผิดพลาดตัวใหม่คือ `fn try_from_os_rng`


## ตัวสร้าง

[`ThreadRng`] ตอนนี้เข้าถึงผ่าน [`rng()`] (เดิมคือ `thread_rng()`)


## ลำดับ

เทรตเดิม `SliceRandom` ถูกแยกออกเป็นสามเทรต: [`IndexedRandom`], [`IndexedMutRandom`] และ [`SliceRandom`] สิ่งนี้ทำให้ฟังก์ชัน `choose` ใช้ได้กับคอนเทนเนอร์คล้าย `Vec` ที่เก็บข้อมูลไม่ต่อเนื่องได้ด้วย แม้ว่าฟังก์ชัน `shuffle` จะยังจำกัดอยู่แค่สไลซ์


## การแจกแจง

โมดูล `rand::distributions` ถูกเปลี่ยนชื่อเป็น [`rand::distr`] เพื่อความกระชับและให้สอดคล้องกับ `rand_distr`

รายการหลายอย่างใน `distr` ก็ถูกเปลี่ยนชื่อหรือย้ายเช่นกัน:

-   Struct `Standard` -> `StandardUniform`
-   Struct `Slice` → `slice::Choose`
-   Struct `EmptySlice` → `slice::Empty`
-   Trait `DistString` → `SampleString`
-   Struct `DistIter` → `Iter`
-   Struct `DistMap` → `Map`
-   Struct `WeightedIndex` → `weighted::WeightedIndex`
-   Enum `WeightedError` → `weighted::Error`

รายการเพิ่มเติมบางอย่างถูกเปลี่ยนชื่อใน `rand_distr`:

-   Struct `weighted_alias::WeightedAliasIndex` → `weighted::WeightedAliasIndex`
-   Trait `weighted_alias::AliasableWeight` → `weighted::AliasableWeight`

การแจกแจง [`StandardUniform`] ไม่รองรับการสุ่มชนิด `Option<T>` อีกต่อไป (สำหรับ `T` ใดๆ)

ชนิด `isize` และ `usize` ไม่ได้รับการรองรับโดย [`Fill`], [`WeightedAliasIndex`] หรือ [`StandardUniform`] อีกต่อไป `isize` ก็ไม่ได้รับการรองรับโดย [`Uniform`] เช่นกัน ส่วน `usize` ยังได้รับการรองรับโดย [`Uniform`] ผ่าน [`UniformUsize`] และตอนนี้ให้ผลลัพธ์ที่พอร์ตได้ข้ามแพลตฟอร์ม 32 และ 64 บิต

คอนสตรัคเตอร์ `fn new`, `fn new_inclusive` ของ [`Uniform`] และ [`UniformSampler`] ตอนนี้คืนค่าเป็น [`Result`] แทนการ panic เมื่ออินพุตไม่ถูกต้อง นอกจากนี้ [`Uniform`] ยังรองรับ [`TryFrom`] (แทน `From`) สำหรับชนิดช่วง


## ฟีเจอร์ nightly

### SIMD

การรองรับ SIMD ตอนนี้มุ่งเป้าไปที่ [`std::simd`]


## ความสามารถในการทำซ้ำ

ดูไฟล์ `CHANGELOG.md` สำหรับรายละเอียดของการเปลี่ยนแปลงที่ทำให้ความสามารถในการทำซ้ำเสียไป ซึ่งกระทบ `rand` และ `rand_distr`


[`Fill`]: https://docs.rs/rand/latest/rand/trait.Fill.html
[`ThreadRng`]: https://docs.rs/rand/latest/rand/rngs/struct.ThreadRng.html
[`ReseedingRng`]: https://docs.rs/rand/latest/rand/rngs/struct.ReseedingRng.html
[`Uniform`]: https://docs.rs/rand/latest/rand/distr/struct.Uniform.html
[`UniformUsize`]: https://docs.rs/rand/latest/rand/distr/uniform/struct.UniformUsize.html
[`WeightedAliasIndex`]: https://docs.rs/rand_distr/latest/rand_distr/weighted_alias/struct.WeightedAliasIndex.html
[`rand_core`]: https://docs.rs/rand_core/
[`rand_distr`]: https://docs.rs/rand_distr/
[`RngCore`]: https://docs.rs/rand_core/latest/rand_core/trait.RngCore.html
[`TryRngCore`]: https://docs.rs/rand_core/latest/rand_core/trait.TryRngCore.html
[`Infallible`]: https://doc.rust-lang.org/std/convert/enum.Infallible.html
[`CryptoRng`]: https://docs.rs/rand/latest/rand/trait.CryptoRng.html
[`TryCryptoRng`]: https://docs.rs/rand/latest/rand/trait.TryCryptoRng.html
[`rng()`]: https://docs.rs/rand/latest/rand/fn.rng.html
[`SliceRandom`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html
[`IndexedRandom`]: https://docs.rs/rand/latest/rand/seq/trait.IndexedRandom.html
[`IndexedMutRandom`]: https://docs.rs/rand/latest/rand/seq/trait.IndexedMutRandom.html
[`StandardUniform`]: https://docs.rs/rand/latest/rand/distr/struct.StandardUniform.html
[`UniformSampler`]: https://docs.rs/rand/latest/rand/distr/uniform/trait.UniformSampler.html
[`Result`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html
[`TryFrom`]: https://doc.rust-lang.org/stable/std/convert/trait.TryFrom.html
[`SeedableRng`]: https://docs.rs/rand_core/latest/rand_core/trait.SeedableRng.html
[`rand::distr`]: https://docs.rs/rand/latest/rand/distr/index.html
[`std::simd`]: https://doc.rust-lang.org/stable/std/simd/index.html
