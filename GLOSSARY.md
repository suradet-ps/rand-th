# พจนานุกรมศัพท์ (Glossary) — The Rust Rand Book ฉบับภาษาไทย

ตารางนี้รวบรวมคำศัพท์เชิงเทคนิคและแนวทางการแปลที่ใช้อย่างสม่ำเสมอตลอดทั้งเล่ม เพื่อให้การแปลทุกบทมีความถูกต้อง ลื่นไหล และเป็นธรรมชาติสำหรับนักพัฒนาซอฟต์แวร์ชาวไทย

| ศัพท์ต้นฉบับ | คำแปลไทย | หมายเหตุ |
|---|---|---|
| randomness | ความสุ่ม (randomness) | |
| random data | ข้อมูลสุ่ม (random data) | ลำดับของไบต์สุ่มที่ทุกค่าเป็นไปได้เท่ากัน |
| random number generator (RNG) | ตัวสร้างเลขสุ่ม (random number generator / RNG) | คง `Rng`, `RngCore`, `RngExt` เมื่อหมายถึงเทรต |
| generator | ตัวสร้าง (generator) | |
| true random number generator (TRNG) | ตัวสร้างเลขสุ่มแท้ (TRNG) | อาศัยกระบวนการธรรมชาติ เช่น การสลายของอะตอม |
| pseudo-random number generator (PRNG) | ตัวสร้างเลขสุ่มเทียม (PRNG) | ตัวสร้างเชิงกำหนดที่เลียนแบบความสุ่ม |
| cryptographically secure PRNG (CSPRNG) | ตัวสร้างเลขสุ่มเทียมเชิงรหัสลับ (CSPRNG) | |
| hardware random number generator (HRNG) | ตัวสร้างเลขสุ่มแบบฮาร์ดแวร์ (HRNG) | |
| seed | ซีด (seed) | ค่าตั้งต้นของตัวสร้างเลขสุ่มเทียม |
| seeding | การซีด (seeding) | การกำหนดสถานะเริ่มต้นจากซีด |
| entropy | เอนโทรปี (entropy) | ปริมาณข้อมูลที่ไม่ทราบค่าในข้อมูลบางส่วน |
| entropy source | แหล่งเอนโทรปี (entropy source) | |
| distribution | การแจกแจง (distribution) | การแจกแจงความน่าจะเป็นของการสุ่มค่า |
| uniform (distribution) | แบบสม่ำเสมอ (uniform) | ช่วงย่อยขนาดเท่ากันมีโอกาสเท่ากัน |
| unbiased | ไม่เอนเอียง (unbiased) | |
| bias | ความเอนเอียง (bias) | |
| sample / sampling | การสุ่มตัวอย่าง (sample / sampling) | |
| sampling with replacement | การสุ่มแบบใส่คืน (with replacement) | |
| sampling without replacement | การสุ่มแบบไม่ใส่คืน (without replacement) | |
| weighted sampling | การสุ่มแบบถ่วงน้ำหนัก (weighted sampling) | |
| shuffle | การสับ (shuffle) | สับลำดับสมาชิกในสไลซ์แบบสุ่ม |
| deterministic | ดีเทอร์มินิสติก / แบบกำหนดได้ | |
| non-deterministic | นอนดีเทอร์มินิสติก / แบบไม่กำหนดได้ | |
| reproducibility | ความสามารถในการทำซ้ำ (reproducibility) | ผลลัพธ์เดิมจากซีดเดิมทุกครั้ง |
| portable / portability | พอร์ตได้ / ความสามารถในการพอร์ต (portability) | ผลลัพธ์เหมือนกันข้ามแพลตฟอร์ม |
| value-breaking | ทำให้ค่าผลลัพธ์เปลี่ยนแปลง (value-breaking) | เปลี่ยนแปลงผลลัพธ์ของกระบวนการดีเทอร์มินิสติกโดยไม่ทำให้ API แตกหัก |
| API-breaking | ทำให้ API แตกหัก (API-breaking) | เปลี่ยนแปลงจนโค้ดเดิมคอมไพล์ไม่ผ่าน |
| major / minor / patch version | เวอร์ชันเมเจอร์ / ไมเนอร์ / แพตช์ | |
| changelog | บันทึกการเปลี่ยนแปลง (changelog) | คง `CHANGELOG.md` เมื่อหมายถึงไฟล์ |
| release | รีลีส (release) | |
| migration guide / porting | คู่มือการย้ายเวอร์ชัน / การพอร์ตโค้ด | |
| deprecated | เลิกใช้ (deprecated) | |
| state | สถานะ (state) | ข้อมูลภายในของตัวสร้าง |
| period / cycle length | คาบ / ความยาววัฏจักร (period / cycle length) | จำนวนค่าที่สร้างได้ก่อนวนซ้ำ |
| thread | เธรด (thread) | |
| thread-local | เธรดโลคอล (thread-local) | หน่วยความจำเฉพาะเธรด |
| parallel | ขนาน (parallel) | |
| worker thread | เธรดทำงาน (worker thread) | |
| work unit | หน่วยงาน (work unit) | |
| stream | สายธาร (stream) | สายธารผลลัพธ์ของ RNG |
| jump / seek | การกระโดด / การเลื่อนตำแหน่ง (jump / seek) | |
| fork | การฟอร์ก (fork) | การแยกโปรเซส |
| side-channel attack | การโจมตีทางช่องทางข้างเคียง (side-channel attack) | |
| forward secrecy | ความลับไปข้างหน้า (forward secrecy) | |
| backtracking resistance | ความต้านทานการย้อนรอย (backtracking resistance) | |
| predictability | ความสามารถในการทำนาย (predictability) | |
| unpredictable | คาดเดาไม่ได้ (unpredictable) | |
| stochastic process | กระบวนการสโทแคสติก (stochastic process) | |
| Monte Carlo | มอนติคาร์โล (Monte Carlo) | |
| cryptographic | เชิงรหัสลับ (cryptographic) | |
| cipher / stream cipher / block cipher | ไซเฟอร์ / ไซเฟอร์แบบสตรีม / ไซเฟอร์แบบบล็อก | |
| key | คีย์ (key) | |
| zeroize | การล้างหน่วยความจำ (zeroize) | คง `zeroize` เมื่อหมายถึงเครต |
| crate | เครต (crate) | |
| trait | เทรต (trait) | |
| struct | struct (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| enum | enum (คงชื่อเดิม) | คำสงวนของภาษา Rust |
| module | โมดูล (module) | |
| field | ฟิลด์ (field) | |
| feature / feature flag | ฟีเจอร์ / ฟีเจอร์แฟล็ก (feature flag) | ตัวเลือกเปิดใช้งานของเครตใน Cargo |
| dependency | ดีเพนเดนซี / ส่วนพึ่งพา (dependency) | |
| array | แอเรย์ (array) | |
| slice | สไลซ์ (slice) | |
| iterator | อิเทอเรเตอร์ (iterator) | |
| closure | โคลเชอร์ (closure) | |
| bound | ข้อกำหนดขอบเขตชนิดข้อมูล / บาวด์ (bound) | เช่น `R: Rng` |
| generic | เจเนอริก (generic) | |
| precision | ความแม่นยำ (precision) | |
| rounding | การปัดเศษ (rounding) | |
| benchmark | การวัดประสิทธิภาพ (benchmark) | |
| hot loop | ลูปที่ถูกเรียกใช้งานบ่อยครั้ง (hot loop) | |
| build | การ build | คงคำว่า build ในบริบทของกระบวนการ build หรือคำสั่ง build (ไม่ใช้ "บิลด์") |
| lazy | แบบเลซี่ (lazy) | การเริ่มต้นเมื่อถูกเรียกใช้งานครั้งแรก |
| unsafe | unsafe (คงเดิม) | คำสงวนของภาษา Rust |
| SIMD | SIMD (คงเดิม) | ชื่อชนิดข้อมูลและฟีเจอร์ |
| WebAssembly / WASM | WebAssembly / WASM (คงเดิม) | |
| operating system (OS) | ระบบปฏิบัติการ (OS) | |
| entropy harvester | ตัวเก็บเกี่ยวเอนโทรปี (entropy harvester) | |
| system under test | ระบบที่กำลังทดสอบ | |

## หลักการทั่วไป

- **ชื่อทางเทคนิค**: ชื่อเครื่องมือ คำสั่ง CLI ตัวเลือก (flag) ชื่อเครต/เทรต/ฟังก์ชัน/ชนิดข้อมูล และ URL **ไม่แปล** เช่น `rand`, `rand_core`, `rand_distr`, `Rng`, `RngCore`, `SeedableRng`, `ChaCha8Rng`, `Uniform`
- **โค้ดทุกบล็อก (` ```...``` `)**: ต้องคงไว้ตามต้นฉบับภาษาอังกฤษทุกตัวอักษร (byte-identical) รวมถึงคอมเมนต์และการเว้นวรรคภายในโค้ด
- **ลิงก์ (Links)**: ทั้ง inline links และ reference links ต้องชี้ไปยัง target เดิมเสมอ เพื่อให้ mdbook build ผ่านและไม่เกิด broken links
- **หัวข้อ (Headings)**: แปลเป็นไทยอย่างเป็นธรรมชาติและกระชับ ยกเว้นหัวข้อที่เป็นชื่อชนิดข้อมูลหรือ API (เช่น `Uniform distributions`, `Rng` และเพื่อนๆ) ซึ่งคงไว้ตามต้นฉบับได้ตามความเหมาะสม
- **Anchor ของลิงก์ภายในเล่ม**: ตรวจสอบกับ HTML ที่ mdbook ทำการ build แล้วเสมอ โดย mdbook จะตัดสระ/วรรณยุกต์ไทยออกจาก slug อัตโนมัติ จึงต้องตรวจสอบด้วย `scripts/check-links.ps1` ทุกครั้ง
