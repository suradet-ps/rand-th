# การแจกแจงแบบสุ่ม

เพื่อความยืดหยุ่นสูงสุดในการสร้างค่าสุ่ม เราจึงนิยามเทรต [`Distribution`] ดังนี้:

```rust,noplayground
# use rand::Rng;
// a producer of data of type T:
pub trait Distribution<T> {
    // the key function:
    fn sample<R: Rng + ?Sized>(&self, rng: &mut R) -> T;

    // a convenience function defined using sample:
    fn sample_iter<R>(self, rng: R) -> rand::distr::Iter<Self, R, T>
    where
        Self: Sized,
        R: Rng,
    {
        // [has a default implementation]
        # todo!()
    }
}
```

การ implement [`Distribution`] คือ*การแจกแจงความน่าจะเป็น*: การแมปจากเหตุการณ์ไปยังความน่าจะเป็น (เช่น สำหรับการทอยลูกเต๋า `P(x = i) = ⅙` หรือสำหรับการแจกแจง Normal ที่มีค่าเฉลี่ย `μ=0`, `P(x > 0) = ½`)

โปรดทราบว่าแม้การแจกแจงความน่าจะเป็นทั้งหมดจะมีคุณสมบัติต่างๆ เช่น ค่าเฉลี่ย ฟังก์ชันความหนาแน่นของความน่าจะเป็น (PDF) และสามารถสุ่มได้โดยการกลับฟังก์ชันการแจกแจงสะสม (CDF) แต่ในที่นี้เราสนใจเพียง*การสุ่มค่าตัวอย่าง* เท่านั้น หากคุณต้องการใช้คุณสมบัติเหล่านี้ คุณอาจเลือกใช้เครต [`statrs`] แทน

Rand ให้การ implement การแจกแจงต่างๆ มากมาย เราครอบคลุมการแจกแจงที่พบบ่อยที่สุดในที่นี้ แต่โปรดดูรายละเอียดทั้งหมดได้ที่โมดูล [`distr`] และเครต [`rand_distr`]

# การแจกแจงแบบสม่ำเสมอ

การแจกแจงแบบที่ชัดเจนที่สุดคือแบบที่เราได้พูดถึงแล้ว: แบบที่ช่วงย่อยขนาดเท่ากันมีโอกาสเท่ากันที่จะมีค่าตัวอย่างถัดไป สิ่งนี้เรียกว่าแบบ *สม่ำเสมอ*

อันที่จริง Rand มีหลายรูปแบบของการแจกแจงนี้ ซึ่งแทนช่วงที่แตกต่างกัน:

-   [`StandardUniform`] ไม่ต้องใช้พารามิเตอร์ และสุ่มค่าแบบสม่ำเสมอตาม
    ชนิดข้อมูล โดย [`Rng::random`] ให้ทางลัดไปยังการแจกแจงนี้
-   [`Uniform`] ถูกกำหนดพารามิเตอร์ด้วย `Uniform::new(low, high)` (รวม `low`
    ไม่รวม `high`) หรือ `Uniform::new_inclusive(low, high)` (รวมทั้งสองค่า)
    และสุ่มค่าแบบสม่ำเสมอภายในช่วงนี้
    [`Rng::random_range`] เป็นเมธอดอำนวยความสะดวกที่นิยามบน
    [`Uniform::sample_single`] ซึ่งปรับแต่งสำหรับการใช้งานแบบสุ่มครั้งเดียว
-   [`Alphanumeric`] สุ่มแบบสม่ำเสมอเฉพาะค่า `char` ในกลุ่ม `0-9A-Za-z`
-   [`Open01`] และ [`OpenClosed01`] ให้ช่วงการสุ่มทางเลือกสำหรับ
    ชนิดข้อมูลทศนิยม (ดูด้านล่าง)

## การสุ่มแบบสม่ำเสมอตามชนิดข้อมูล

มาดูการแจกแจงตามชนิดข้อมูลกัน:

-   สำหรับ `bool` [`StandardUniform`] สุ่มแต่ละค่าด้วยความน่าจะเป็น 50%
-   สำหรับ `Option<T>` การแจกแจง [`StandardUniform`] สุ่ม `None` ด้วย
    ความน่าจะเป็น 50% มิฉะนั้นจะสุ่ม `Some(value)` ตามชนิดข้อมูลของมัน
-   สำหรับจำนวนเต็ม (`u8` จนถึง `u128`, `usize` และชนิด `i*` ต่างๆ)
    [`StandardUniform`] สุ่มจากค่าที่เป็นไปได้ทั้งหมด ขณะที่
    [`Uniform`] สุ่มจากช่วงที่กำหนดพารามิเตอร์
-   สำหรับ `NonZeroU8` และชนิด "ไม่เป็นศูนย์" อื่นๆ [`StandardUniform`] สุ่มแบบสม่ำเสมอ
    จากค่าที่ไม่เป็นศูนย์ทั้งหมด (วิธีปฏิเสธ)
-   ชนิดจำนวนเต็ม `Wrapping<T>` ถูกสุ่มเหมือนชนิดจำนวนเต็มที่สอดคล้องกัน
    โดยการแจกแจง [`StandardUniform`]
-   สำหรับทศนิยม (`f32`, `f64`)

    -   [`StandardUniform`] สุ่มจากช่วงครึ่งเปิด `[0, 1)` ด้วยความแม่นยำ 24 หรือ 53
        บิต (สำหรับ `f32` และ `f64` ตามลำดับ)
    -   [`OpenClosed01`] สุ่มจากช่วงครึ่งเปิด `(0, 1]` ด้วยความแม่นยำ 24 หรือ
        53 บิต
    -   [`Open01`] สุ่มจากช่วงเปิด `(0, 1)` ด้วยความแม่นยำ 23 หรือ 52 บิต
    -   [`Uniform`] สุ่มจากช่วงที่กำหนดด้วยความแม่นยำ 23 หรือ 52 บิต
-   สำหรับชนิด `char` การแจกแจง [`StandardUniform`] สุ่มจากรหัส Unicode
    ทั้งหมดที่มีอย่างสม่ำเสมอ; หลายค่าในนั้นอาจพิมพ์ไม่ได้ (ขึ้นอยู่กับการรองรับฟอนต์) [`Alphanumeric`] สุ่มจาก
    เฉพาะ a-z, A-Z และ 0-9 แบบสม่ำเสมอ
-   สำหรับทูเพิลและแอเรย์ แต่ละสมาชิกถูกสุ่มตามข้างต้นในกรณีที่รองรับ
    การแจกแจง [`StandardUniform`] และ [`Uniform`] ต่างรองรับชนิดข้อมูลเหล่านี้บางส่วน
    (ไม่เกินทูเพิล 12 สมาชิกและแอเรย์ 32 สมาชิก)
    ซึ่งรวมถึงทูเพิลว่าง `()` และแอเรย์ว่างด้วย
    เมื่อใช้ `rustc` ≥ 1.51 ให้เปิดใช้ฟีเจอร์ `min_const_gen` เพื่อรองรับ
    แอเรย์ที่มีสมาชิกมากกว่า 32 ตัว
-   สำหรับชนิด SIMD แต่ละสมาชิกถูกสุ่มตามข้างต้น สำหรับ [`StandardUniform`] และ
    [`Uniform`] (สำหรับแบบหลัง พารามิเตอร์ `low` และ `high` *ก็เป็น* ชนิด
    SIMD เช่นกัน จึงสุ่มจากหลายช่วงพร้อมกันได้อย่างมีประสิทธิผล) การรองรับ SIMD
    ต้องใช้ฟีเจอร์แฟล็ก `simd_support` และ `rustc` เวอร์ชัน nightly
-   สำหรับ enum คุณต้อง implement การสุ่มแบบสม่ำเสมอเอง ตัวอย่างเช่น
    คุณอาจใช้วิธีต่อไปนี้:
    ```rust,noplayground
    # use rand::{Rng, RngExt, distr::{Distribution, StandardUniform}};
    pub enum Food {
        Burger,
        Pizza,
        Kebab,
    }

    impl Distribution<Food> for StandardUniform {
        fn sample<R: Rng + ?Sized>(&self, rng: &mut R) -> Food {
            let index: u8 = rng.random_range(0..3);
            match index {
                0 => Food::Burger,
                1 => Food::Pizza,
                2 => Food::Kebab,
                _ => unreachable!(),
            }
        }
    }
    ```

# การแจกแจงแบบไม่สม่ำเสมอ

เครต [`rand`] ให้เฉพาะการแจกแจงแบบไม่สม่ำเสมอสองแบบเท่านั้น:

-   การแจกแจง [`Bernoulli`] เพียงแค่สร้างค่าบูลีน โดยความน่าจะเป็นที่จะสุ่มได้ `true`
    นั้นเป็นค่าคงที่ (`Bernoulli::new(0.5)`) หรืออัตราส่วน
    (`Bernoulli::from_ratio(1, 6)`)
-   การแจกแจง [`WeightedIndex`] ใช้สุ่มจากลำดับของค่าที่ถ่วงน้ำหนักได้
    ดูส่วน [Sequences]

การแจกแจงแบบไม่สม่ำเสมออื่นๆ อีกมากมีให้ในเครต [`rand_distr`]

### จำนวนเต็ม

การแจกแจง [`Binomial`] เกี่ยวข้องกับ [`Bernoulli`] ตรงที่มันจำลองการทดลองอิสระ `n` ครั้ง แต่ละครั้งมีความน่าจะเป็น `p` ที่จะสำเร็จ แล้วนับจำนวนครั้งที่สำเร็จ

โปรดทราบว่าสำหรับ `n` ที่มาก การ implement การแจกแจง [`Binomial`] จะเร็วกว่าการสุ่ม `n` ครั้งทีละครั้งมาก

การแจกแจง [`Poisson`] แสดงจำนวนเหตุการณ์ที่คาดว่าจะเกิดขึ้นภายในช่วงเวลาคงที่ โดยที่เหตุการณ์เกิดขึ้นด้วยอัตราคงที่ λ การสุ่มจากการแจกแจง [`Poisson`] จะสร้างค่า `Float` เพราะการคำนวณการสุ่มใช้ `Float` และเราเลือกที่จะให้ผู้ใช้เป็นผู้ตัดสินใจเรื่องชนิดจำนวนเต็ม รวมถึงการแปลงที่เกี่ยวข้องซึ่งอาจสูญเสียข้อมูลและ panic ได้ด้วยตนเอง ตัวอย่างเช่น ค่า `u64` หาได้ด้วย `rng.sample(Poisson) as u64`

โปรดทราบว่าการแปลงทศนิยมเป็นจำนวนเต็มที่อยู่นอกช่วงด้วย `as` ให้ผลเป็นพฤติกรรมที่นิยามไม่ได้สำหรับ Rust <1.45 และเป็นการแปลงแบบอิ่มตัวสำหรับ Rust >=1.45

## การแจกแจงแบบไม่สม่ำเสมอต่อเนื่อง

การแจกแจงต่อเนื่องจำลองตัวอย่างที่ดึงมาจากเส้นจำนวนจริง ℝ หรือในบางกรณีเป็นจุดจากมิติที่สูงกว่า (ℝ², ℝ³ ฯลฯ) เรามีการ implement สำหรับเอาต์พุต `f64` และ `f32` ในกรณีส่วนใหญ่ แม้ว่าปัจจุบันการ implement สำหรับ `f32` เพียงลดความแม่นยำของตัวอย่าง `f64`

การแจกแจงแบบเอกซ์โพเนนเชียล [`Exp`] จำลองเวลาจนถึงการสลายตัว โดยสมมติว่าอัตราการสลายตัวคงที่ (กล่าวคือ การสลายตัวแบบเอกซ์โพเนนเชียล)

การแจกแจง [`Normal`] (หรือที่รู้จักกันในชื่อการแจกแจงแบบเกาส์) จำลองการสุ่มจากการแจกแจง Normal ("เส้นโค้งระฆัง") ด้วยค่าเฉลี่ยและส่วนเบี่ยงเบนมาตรฐานที่กำหนด [`LogNormal`] เกี่ยวข้องกัน: สำหรับตัวอย่าง `X` จากการแจกแจงล็อกนอร์มัล `log(X)` จะแจกแจงแบบนอร์มัล; สิ่งนี้ทำให้การแจกแจงนอร์มัล "เบ้" เพื่อหลีกเลี่ยงค่าลบและมีหางบวกยาว

การแจกแจง [`UnitCircle`] และ [`UnitSphere`] จำลองการสุ่มแบบสม่ำเสมอจากเส้นรอบวงกลมหรือพื้นผิวทรงกลม

การแจกแจง [`Cauchy`] (หรือที่รู้จักกันในชื่อการแจกแจง Lorentz) คือการแจกแจงของจุดตัดแกน x ของรังสีจากจุด `(x0, γ)` ที่มีมุมแจกแจงแบบสม่ำเสมอ

การแจกแจง [`Beta`] เป็นการแจกแจงความน่าจะเป็นแบบสองพารามิเตอร์ ซึ่งค่าเอาต์พุตอยู่ระหว่าง 0 กับ 1 การแจกแจง [`Dirichlet`] เป็นการวางนัยทั่วไปเป็นจำนวนพารามิเตอร์บวกใดๆ

[Sequences]: guide-seq.html
[`Distribution`]: https://docs.rs/rand/latest/rand/distr/trait.Distribution.html
[`distr`]: https://docs.rs/rand/latest/rand/distr/
[`rand`]: https://docs.rs/rand/
[`rand_distr`]: https://docs.rs/rand_distr/
[`Rng::random_range`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_range
[`random`]: https://docs.rs/rand/latest/rand/fn.random.html
[`Rng::random_bool`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_bool
[`Rng::random_ratio`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random_ratio
[`Rng::random`]: https://docs.rs/rand/latest/rand/trait.Rng.html#method.random
[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`StandardUniform`]: https://docs.rs/rand/latest/rand/distr/struct.StandardUniform.html
[`Uniform`]: https://docs.rs/rand/latest/rand/distr/struct.Uniform.html
[`Uniform::sample_single`]: https://docs.rs/rand/latest/rand/distr/struct.Uniform.html#method.sample_single
[`Alphanumeric`]: https://docs.rs/rand/latest/rand/distr/struct.Alphanumeric.html
[`Open01`]: https://docs.rs/rand/latest/rand/distr/struct.Open01.html
[`OpenClosed01`]: https://docs.rs/rand/latest/rand/distr/struct.OpenClosed01.html
[`Bernoulli`]: https://docs.rs/rand/latest/rand/distr/struct.Bernoulli.html
[`Binomial`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Binomial.html
[`Exp`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Exp.html
[`Normal`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Normal.html
[`LogNormal`]: https://docs.rs/rand_distr/latest/rand_distr/struct.LogNormal.html
[`UnitCircle`]: https://docs.rs/rand_distr/latest/rand_distr/struct.UnitCircle.html
[`UnitSphere`]: https://docs.rs/rand_distr/latest/rand_distr/struct.UnitSphere.html
[`Cauchy`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Cauchy.html
[`Poisson`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Poisson.html
[`Beta`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Beta.html
[`Dirichlet`]: https://docs.rs/rand_distr/latest/rand_distr/struct.Dirichlet.html
[`statrs`]: https://github.com/statrs-dev/statrs/
[`WeightedIndex`]: https://docs.rs/rand/latest/rand/distr/weighted/struct.WeightedIndex.html
