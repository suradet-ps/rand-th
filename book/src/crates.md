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

## ส่วนต่อประสาน

[`rand_core`] นิยาม [`RngCore`] และเทรตหลักอื่นๆ รวมถึงตัวช่วยหลายอย่างสำหรับการ implement RNG

เครต [`getrandom`] ให้ API ระดับต่ำสำหรับเข้าถึงแหล่งเลขสุ่มเฉพาะแพลตฟอร์ม

## ตัวสร้างเลขสุ่มเทียม

เครตต่อไปนี้ implement ตัวสร้างเลขสุ่มเทียม
(ดู [ตัวสร้างเลขสุ่มของเรา](guide-rngs.md)):

-   [`chacha20`] ให้ตัวสร้างที่ใช้ไซเฟอร์ ChaCha
    (ถูกใช้ภายในโดย [`rand`] ผ่านฟีเจอร์ `chacha`)
-   [`rand_chacha`] ก็ให้ตัวสร้างแบบไซเฟอร์ ChaCha เช่นกัน
    (รุ่นก่อนของ `chacha20` ในหน้าที่นี้ เก็บไว้เพื่อความเข้ากันได้)
-   [`rand_hc`] implement ตัวสร้างที่ใช้ไซเฟอร์ HC-128
-   [`rand_isaac`] implement ตัวสร้าง ISAAC
-   [`rand_pcg`] implement ตัวสร้าง PCG บางส่วน
-   [`rand_sfc`] implement ตัวสร้าง SFC (Small Fast Counter)
-   [`rand_xorshift`] implement ตัวสร้าง Xorshift พื้นฐาน
-   [`rand_xoshiro`] implement ตัวสร้าง SplitMix และ Xoshiro

เป็นข้อยกเว้นว่า [`SmallRng`] ถูก implement โดยตรงใน [`rand`]

นอกจากนี้ [`rand_jitter`] ให้แหล่งเอนโทรปีแบบ jitter และ
[`rand_seeder`] สร้างซีดจากข้อมูลที่แฮชได้ตามอำเภอใจ (ใช้เป็นส่วนประกอบใน [การซีด RNG](guide-seeding.md))

## rand (เครตหลัก)

เครต [`rand`] ถูกออกแบบมาเพื่อการใช้งานฟังก์ชันเลขสุ่มทั่วไปได้อย่างง่ายดาย
ซึ่งมีหลายแง่มุมดังนี้:

-   โมดูล [`rngs`] ให้ตัวสร้างที่สะดวกใช้งานหลายตัว
-   โมดูล [`distr`] เกี่ยวข้องกับการสุ่มค่าสุ่ม
-   โมดูล [`seq`] เกี่ยวข้องกับการสุ่มจากลำดับและการสับลำดับ
-   เทรต [`Rng`] ให้เมธอดอำนวยความสะดวกหลายอย่างสำหรับสร้างค่าสุ่ม
-   ฟังก์ชัน [`random`] ให้การสร้างค่าแบบเรียกครั้งเดียวที่สะดวก

## การแจกแจง

เครต [`rand`] implement เฉพาะการสุ่มจากการแจกแจงเลขสุ่มที่พบบ่อยที่สุดเท่านั้น
นั่นคือการสุ่มแบบสม่ำเสมอและการสุ่มแบบถ่วงน้ำหนัก สำหรับอย่างอื่นนั้น

-   [`rand_distr`] ให้การสุ่มที่รวดเร็วจากการแจกแจงรูปแบบอื่นๆ มากมาย
    รวมถึง Normal (Gauss), Binomial, Poisson, UnitCircle และอื่นๆ อีกมาก
-   [`statrs`] เป็นพอร์ตของไลบรารี Math.NET จากภาษา C# ซึ่ง implement
    การแจกแจงหลายอย่างเดียวกัน (มากกว่า/น้อยกว่าเล็กน้อย) พร้อมฟังก์ชัน PDF และ CDF
    ฟังก์ชันพิเศษ *error*, *beta*, *gamma* และ *logistic* รวมถึงยูทิลิตี้อีกเล็กน้อย
    (เพื่อความชัดเจน [`statrs`] ไม่ได้เป็นส่วนหนึ่งของไลบรารี Rand)


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
