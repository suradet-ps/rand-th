# เอกสาร

### สไตล์

เอกสารทั้งหมดจัดทำเป็นภาษาอังกฤษ โดยไม่ได้จำกัดว่าจะต้องใช้สำเนียงหรือรูปแบบภาษาของภูมิภาคใดเป็นพิเศษ

เอกสารควรจะเข้าถึงและเข้าใจได้ง่ายสำหรับผู้อ่านทุกระดับ ทั้งชาว Rustacean ผู้ช่ำชองและผู้ที่เพิ่งเริ่มต้นใช้งาน รวมถึงผู้ที่มีความเชี่ยวชาญด้านการสร้างแบบจำลองทางสถิติหรือวิทยาการรหัสลับ ตลอดจนผู้ที่เพิ่งเริ่มศึกษาศาสตร์เหล่านี้ เนื่องจากเป็นเรื่องยากที่จะเขียนเอกสารฉบับเดียวให้ตอบโจทย์ผู้อ่านทุกคนได้อย่างสมบูรณ์แบบ เราจึงมุ่งเน้นการเขียนเอกสารเชิงเทคนิคที่กระชับ พร้อมทั้งใส่ลิงก์อ้างอิงไปยังบทความหรือแหล่งข้อมูลเพิ่มเติมสำหรับกลุ่มผู้อ่านเฉพาะทาง

## เอกสาร API

### เครตของ Rand

ขอแนะนำให้ใช้คอมไพเลอร์ Rust เวอร์ชัน nightly เพื่อให้เครื่องมือสามารถประมวลผลลิงก์ต่างๆ ได้อย่างถูกต้อง

หากต้องการ build เอกสาร API ของทุกเครตในคลังเก็บโค้ด
[rust-random/rand](https://github.com/rust-random/rand) ให้รันคำสั่ง:

```sh
# Optionally, enable some unstable but widely used doc features:
export RUSTDOCFLAGS="--cfg docsrs -Zunstable-options --generate-link-to-definition"

# Build doc for all crates in the workspace:
cargo doc --workspace --no-deps --all-features --open
```
(หรือตรวจสอบไฟล์ `Cargo.toml` ใต้หัวข้อ `[package.metadata.docs.rs]` ซึ่งอาจมีการระบุการตั้งค่าเฉพาะของ workspace หรือแต่ละเครตไว้)

บนระบบปฏิบัติการ Linux คุณสามารถตั้งค่าให้ rebuild เอกสารใหม่อัตโนมัติหลังการแก้ไขทุกครั้งได้ง่ายๆ:
```sh
while inotifywait -r -e close_write src/ rand_*/; do cargo doc; done
```

หลังจากแก้ไขเอกสาร API แล้ว ขอแนะนำให้ทดสอบโค้ดตัวอย่างในเอกสารด้วยคำสั่ง:

```sh
cargo test --doc
```

### เครต Getrandom

คลังเก็บโค้ด [rust-random/getrandom](https://github.com/rust-random/getrandom)
มีเพียงเครตเดียว ดังนั้นการสั่ง `cargo doc` เพียงอย่างเดียวก็เพียงพอแล้ว

## เอกสารเสริม

### ไฟล์ README

ไฟล์ README จะประกอบด้วยบทนำสั้นๆ เกี่ยวกับเครต ป้าย shields ลิงก์ที่เป็นประโยชน์ เอกสารอธิบายฟีเจอร์แฟล็ก ข้อมูลสัญญาอนุญาต ตลอดจนตัวอย่างการใช้งาน

โดยทั่วไปแล้วไฟล์เหล่านี้จะไม่มีการทดสอบต่อเนื่อง
อย่างไรก็ตาม ในจุดที่มีตัวอย่างโค้ดรวมอยู่ด้วย (ปัจจุบันมีเฉพาะในเครต `rand_jitter`) เราจะเปิดใช้งานการทดสอบอย่างต่อเนื่องผ่าน `doc_comment` (ดูรายละเอียดได้ที่
[lib.rs:62 เป็นต้นไป](https://github.com/rust-random/rngs/blob/master/rand_jitter/src/lib.rs#L62))

### ไฟล์ CHANGELOG

รูปแบบของบันทึกการเปลี่ยนแปลงจะอ้างอิงตามรูปแบบ
[Keep a Changelog](http://keepachangelog.com/en/1.0.0/)

การเปลี่ยนแปลงสำคัญทั้งหมดที่ถูก merge เข้ามานับตั้งแต่การเปิดตัวเวอร์ชันล่าสุด จะต้องถูกระบุไว้ภายใต้หัวข้อ `[Unreleased]` ที่ด้านบนสุดของบันทึก

### หนังสือเล่มนี้

ซอร์สโค้ดของหนังสือเล่มนี้อยู่ในคลัง
[rust-random/book](https://github.com/rust-random/book)
หนังสือเล่มนี้ถูกสร้างด้วย mdbook ซึ่งช่วยให้การ build และทดสอบเป็นเรื่องง่าย:

```sh
cargo install mdbook --version "^0.4"

mdbook build --open
mdbook test

# To automatically rebuild after any changes:
mdbook watch
```

โปรดทราบว่าลิงก์ในหนังสือเป็นแบบสัมพัทธ์และออกแบบให้ทำงานใน
[หนังสือที่เผยแพร่แล้ว](https://rust-random.github.io/book/) หากคุณ build หนังสือในเครื่อง คุณอาจต้องการสร้าง symbolic link ชี้ไปยังเอกสาร API ที่คุณ build ไว้ในเครื่องดังนี้:
```sh
ln -s ../rand/target/doc rand
```

[docs#204]: https://github.com/rust-lang/docs.rs/issues/204
