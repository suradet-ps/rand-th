# ลำดับ

Rand implement การดำเนินการแบบสุ่มที่พบบ่อยบนลำดับหลายอย่างผ่านเทรต
[`IteratorRandom`] และ [`SliceRandom`]

## การสร้างดัชนี

การสุ่ม:

-   ดัชนีเดียวภายในช่วงที่กำหนด ใช้ [`Rng::random_range`]
-   ดัชนีที่แตกต่างกันหลายตัวจาก `0..length` ใช้ [`index::sample`]
-   ดัชนีที่แตกต่างกันหลายตัวจาก `0..length` พร้อมน้ำหนัก ใช้ [`index::sample_weighted`]

## การสับ

การสับสไลซ์:

-   [`SliceRandom::shuffle`]: สับสไลซ์ทั้งตัว
-   [`SliceRandom::partial_shuffle`]: สับบางส่วน; มีประโยชน์ในการดึงสมาชิก
    `amount` ตัวแบบสุ่มในลำดับแบบสุ่ม

## การสุ่มตัวอย่าง

ต่อไปนี้เป็นวิธีที่สะดวกในการสุ่มค่าจากสไลซ์หรืออิเทอเรเตอร์:

-   [`SliceRandom::choose`]: สุ่มสมาชิกหนึ่งตัวจากสไลซ์ (แบบ ref)
-   [`SliceRandom::choose_mut`]: สุ่มสมาชิกหนึ่งตัวจากสไลซ์ (แบบ ref mut)
-   [`SliceRandom::choose_multiple`]: สุ่มสมาชิกที่แตกต่างกันหลายตัวจากสไลซ์ (คืนอิเทอเรเตอร์ของอ้างอิงถึงสมาชิก)
-   [`IteratorRandom::choose`]: สุ่มสมาชิกหนึ่งตัวจากอิเทอเรเตอร์ (แบบ value)
-   [`IteratorRandom::choose_stable`]: สุ่มสมาชิกหนึ่งตัวจากอิเทอเรเตอร์ (แบบ value) โดยการเรียกใช้ RNG ไม่ได้รับผลกระทบจาก [`size_hint`] ของอิเทอเรเตอร์
-   [`IteratorRandom::choose_multiple_fill`]: สุ่มสมาชิกหลายตัว แล้วใส่ลงในบัฟเฟอร์
-   [`IteratorRandom::choose_multiple`]: สุ่มสมาชิกหลายตัว แล้วคืน [`Vec`]

โปรดทราบว่าการดำเนินการกับอิเทอเรเตอร์มักมีประสิทธิภาพน้อยกว่าการดำเนินการกับสไลซ์

## การสุ่มแบบถ่วงน้ำหนัก

ตัวอย่างเช่น การสุ่มแบบถ่วงน้ำหนักอาจใช้จำลองสีของลูกแก้วที่สุ่มจากถังที่บรรจุสีเขียว 5 ลูก สีแดง 15 ลูก และสีน้ำเงิน 80 ลูก

### แบบใส่คืน

การสุ่ม*แบบใส่คืน*หมายความว่าค่าที่สุ่มได้ (ลูกแก้ว) ถูกใส่คืน (ดังนั้น ความน่าจะเป็นในการสุ่มแต่ละชนิดจึงไม่ได้รับผลกระทบจากการสุ่ม)

สิ่งนี้ถูก implement โดยการแจกแจงต่อไปนี้:

-   [`WeightedIndex`] ตั้งค่าเริ่มต้นได้เร็วและสุ่มด้วย `O(log N)`
-   [`WeightedAliasIndex`] ตั้งค่าเริ่มต้นได้ช้าและสุ่มด้วย `O(1)` จึง*อาจ*
    เร็วกว่าเมื่อมีจำนวนตัวอย่างมาก

เพื่อความสะดวก คุณอาจใช้:

-   [`SliceRandom::choose_weighted`]
-   [`SliceRandom::choose_weighted_mut`]

### แบบไม่ใส่คืน

การสุ่ม*แบบไม่ใส่คืน*หมายความว่าการสุ่มเปลี่ยนแปลงการแจกแจง เนื่องจากเทรต [`Distribution`] สร้างขึ้นบนแนวคิดของการแจกแจงที่ไม่เปลี่ยนแปลง เราจึงมีสิ่งต่อไปนี้:

-   [`SliceRandom::choose_multiple_weighted`]: สุ่มค่าที่แตกต่างกัน `amount` ตัว
    จากสไลซ์พร้อมน้ำหนัก
-   [`index::sample_weighted`]: สุ่มดัชนีที่แตกต่างกัน `amount` ตัวจากช่วงพร้อม
    น้ำหนัก
-   implement เอง: ดูหัวข้อใน [กระบวนการสุ่ม](guide-process.html#การสุมแบบไมใสคืน)

[`Distribution`]: https://docs.rs/rand/latest/rand/distr/trait.Distribution.html
[`IteratorRandom`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html
[`SliceRandom`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html
[`WeightedIndex`]: https://docs.rs/rand_distr/latest/rand_distr/weighted/struct.WeightedIndex.html
[`WeightedAliasIndex`]: https://docs.rs/rand_distr/latest/rand_distr/weighted/struct.WeightedAliasIndex.html
[`SliceRandom::choose`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose
[`SliceRandom::choose_mut`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_mut
[`SliceRandom::choose_multiple`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_multiple
[`IteratorRandom::choose`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html#method.choose
[`IteratorRandom::choose_stable`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html#method.choose_stable
[`IteratorRandom::choose_multiple`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html#method.choose_multiple
[`IteratorRandom::choose_multiple_fill`]: https://docs.rs/rand/latest/rand/seq/trait.IteratorRandom.html#method.choose_multiple_fill
[`SliceRandom::choose_weighted`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_weighted
[`SliceRandom::choose_weighted_mut`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_weighted_mut
[`SliceRandom::choose_multiple_weighted`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.choose_multiple_weighted
[`SliceRandom::shuffle`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.shuffle
[`SliceRandom::partial_shuffle`]: https://docs.rs/rand/latest/rand/seq/trait.SliceRandom.html#tymethod.partial_shuffle
[`Rng::random_range`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_range
[`index::sample`]: https://docs.rs/rand/latest/rand/seq/index/fn.sample.html
[`index::sample_weighted`]: https://docs.rs/rand/latest/rand/seq/index/fn.sample_weighted.html
[`size_hint`]: https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html#method.size_hint
[`Vec`]: https://doc.rust-lang.org/stable/std/vec/struct.Vec.html
