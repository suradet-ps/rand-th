# ตระกูลเครต

<pre><code class="language-plain">                                           ┌ <a href="https://docs.rs/statrs/">statrs</a>
                                           ├ <a href="https://docs.rs/rand_distr/">rand_distr</a>
          <a href="https://docs.rs/rand_core/">rand_core</a> ┬───────────────┬ <a href="https://docs.rs/rand/">rand</a> ┘
                    ├ <a href="https://docs.rs/chacha20/">chacha20</a> ─────┤
                    ├ <a href="https://docs.rs/getrandom/">getrandom</a> ────┘
                    ├ <a href="https://docs.rs/rand_chacha/">rand_chacha</a>
                    ├ <a href="https://docs.rs/rand_hc/">rand_hc</a>
                    ├ <a href="https://docs.rs/rand_isaac/">rand_isaac</a>
                    ├ <a href="https://docs.rs/rand_jitter/">rand_jitter</a>
                    ├ <a href="https://docs.rs/rand_pcg/">rand_pcg</a>
                    ├ <a href="https://docs.rs/rand_sfc/">rand_sfc</a>
                    ├ <a href="https://docs.rs/rand_seeder/">rand_seeder</a>
                    ├ <a href="https://docs.rs/rand_xorshift/">rand_xorshift</a>
                    └ <a href="https://docs.rs/rand_xoshiro/">rand_xoshiro</a>
</code></pre>

## อินเทอร์เฟซ

[`rand_core`] ทำหน้าที่นิยาม [`RngCore`] และเทรตหลักอื่นๆ ตลอดจนเครื่องมือช่วยอำนวยความสะดวกสำหรับผู้ที่ต้องการอิมพลีเมนต์ RNG ขึ้นมาเอง

เครต [`getrandom`] มอบ API ระดับต่ำสำหรับเชื่อมต่อกับแหล่งสร้างเลขสุ่มเฉพาะของแต่ละแพลตฟอร์ม

## ตัวสร้างเลขสุ่มเทียม

เครตต่อไปนี้ทำหน้าที่อิมพลีเมนต์ตัวสร้างเลขสุ่มเทียม (PRNG) หลากหลายรูปแบบ
(ดูเพิ่มเติมได้ที่ [ตัวสร้างเลขสุ่มของเรา](guide-rngs.md)):

-   [`chacha20`] มอบตัวสร้างที่ใช้อัลกอริทึมการเข้ารหัส ChaCha
    (ถูกเรียกใช้เป็นการภายในโดย [`rand`] ผ่านฟีเจอร์แฟล็ก `chacha`)
-   [`rand_chacha`] มอบตัวสร้างที่ใช้ ChaCha เช่นกัน
    (เป็นเวอร์ชันดั้งเดิมของ `chacha20` สำหรับงานนี้ ซึ่งคงไว้เพื่อความเข้ากันได้ย้อนหลัง)
-   [`rand_hc`] อิมพลีเมนต์ตัวสร้างที่ใช้อัลกอริทึมการเข้ารหัส HC-128
-   [`rand_isaac`] อิมพลีเมนต์ตัวสร้างตระกูล ISAAC
-   [`rand_pcg`] อิมพลีเมนต์ตัวสร้างตระกูล PCG สำหรับบางโมเดลที่คัดสรรมา
-   [`rand_sfc`] อิมพลีเมนต์ตัวสร้างตระกูล SFC (Small Fast Counter)
-   [`rand_xorshift`] อิมพลีเมนต์ตัวสร้าง Xorshift ขั้นพื้นฐาน
-   [`rand_xoshiro`] อิมพลีเมนต์ตัวสร้าง SplitMix และ Xoshiro

โดยมีข้อยกเว้นคือ [`SmallRng`] ซึ่งถูกอิมพลีเมนต์ไว้ใน [`rand`] โดยตรง

นอกจากนี้ [`rand_jitter`] ยังมอบแหล่งเอนโทรปีที่อิงจากความผันผวนของรอบสัญญาณนาฬิกา CPU (jitter) และ
[`rand_seeder`] ทำหน้าที่แปลงข้อมูลใดๆ ที่สามารถแฮชได้มาสร้างเป็นค่าซีด (ใช้เป็นองค์ประกอบสำคัญใน [การซีด RNG](guide-seeding.md))

## rand (เครตหลัก)

เครต [`rand`] ได้รับการออกแบบขึ้นเพื่อให้สามารถเรียกใช้งานฟังก์ชันเกี่ยวกับเลขสุ่มทั่วไปได้อย่างง่ายดาย
โดยแบ่งความสามารถออกเป็นหลายด้านดังนี้:

-   โมดูล [`rngs`] มีตัวสร้างเลขสุ่มสำเร็จรูปที่สะดวกต่อการใช้งานทั่วไป
-   โมดูล [`distr`] รับผิดชอบการสุ่มค่าตามการแจกแจงทางสถิติ
-   โมดูล [`seq`] รับผิดชอบการสุ่มเลือกสมาชิกจากลำดับข้อมูลและการสับเปลี่ยนลำดับ
-   เทรต [`Rng`] รวบรวมเมธอดอำนวยความสะดวกหลากหลายรูปแบบสำหรับการสร้างค่าสุ่ม
-   ฟังก์ชัน [`random`] ช่วยให้สุ่มสร้างค่าได้ง่ายๆ ในการเรียกใช้เพียงคำสั่งเดียว

## การแจกแจง

เครต [`rand`] มุ่งเน้นอิมพลีเมนต์เฉพาะการสุ่มจากการแจกแจงที่เป็นที่นิยมที่สุดเท่านั้น
ได้แก่ การสุ่มแบบสม่ำเสมอ (Uniform) และการสุ่มแบบถ่วงน้ำหนัก (Weighted sampling) ส่วนการแจกแจงรูปแบบอื่นๆ มีเครตแยกเฉพาะรองรับ:

-   [`rand_distr`] มอบการสุ่มค่าที่รวดเร็วจากการแจกแจงทางสถิติที่หลากหลาย
    รวมถึง Normal (Gauss), Binomial, Poisson, UnitCircle และอื่นๆ อีกมากมาย
-   [`statrs`] เป็นไลบรารีที่พอร์ตมาจาก Math.NET ของภาษา C# ซึ่งอิมพลีเมนต์
    การแจกแจงหลายชนิดคล้ายคลึงกัน (อาจมีมากกว่าหรือน้อยกว่าในบางฟังก์ชัน) ควบคู่ไปกับฟังก์ชันความหนาแน่นความน่าจะเป็น (PDF) และฟังก์ชันการแจกแจงสะสม (CDF)
    ฟังก์ชันพิเศษทางคณิตศาสตร์อย่าง *error*, *beta*, *gamma* และ *logistic* ตลอดจนเครื่องมืออำนวยความสะดวกอื่นๆ
    (ทั้งนี้ เพื่อความชัดเจน [`statrs`] ไม่ได้เป็นส่วนหนึ่งของไลบรารี Rand อย่างเป็นทางการ)


[`rand_core`]: https://docs.rs/rand_core/
[`rand`]: https://docs.rs/rand/
[`rand_distr`]: https://docs.rs/rand_distr/
[`statrs`]: https://docs.rs/statrs/
[`getrandom`]: https://docs.rs/getrandom/
[`chacha20`]: https://docs.rs/chacha20/
[`rand_pcg`]: https://docs.rs/rand_pcg/
[`rand_xoshiro`]: https://docs.rs/rand_xoshiro/
[`log`]: https://docs.rs/log/
[`serde`]: https://serde.rs/
[`rand_chacha`]: https://docs.rs/rand_chacha/
[`rand_hc`]: https://docs.rs/rand_hc/
[`rand_isaac`]: https://docs.rs/rand_isaac/
[`rand_jitter`]: https://docs.rs/rand_jitter/
[`rand_sfc`]: https://docs.rs/rand_sfc/
[`rand_seeder`]: https://docs.rs/rand_seeder/
[`rand_xorshift`]: https://docs.rs/rand_xorshift/

[`RngCore`]: https://docs.rs/rand_core/latest/rand_core/trait.RngCore.html

[`rngs`]: https://docs.rs/rand/latest/rand/rngs/
[`distr`]: https://docs.rs/rand/latest/rand/distr/
[`seq`]: https://docs.rs/rand/latest/rand/seq/
[`Rng`]: https://docs.rs/rand/latest/rand/trait.Rng.html
[`random`]: https://docs.rs/rand/latest/rand/fn.random.html

[`SmallRng`]: https://docs.rs/rand/latest/rand/rngs/struct.SmallRng.html
