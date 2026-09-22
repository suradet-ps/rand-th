# เอกสาร

### สไตล์

เอกสารทั้งหมดเป็นภาษาอังกฤษ แต่ไม่จำกัดสำเนียงใดเป็นพิเศษ

เอกสารควรเข้าถึงได้สำหรับผู้อ่านหลายกลุ่ม ทั้งชาว Rustacean ผู้ช่ำชองและผู้มาใหม่ รวมถึงผู้ที่มีประสบการณ์ด้านการสร้างแบบจำลองทางสถิติหรือการเข้ารหัส ตลอดจนผู้ที่เพิ่งเริ่มเรียนรู้เรื่องเหล่านี้ เนื่องจากบ่อยครั้งเป็นไปไม่ได้ที่จะเขียนเอกสารแบบเดียวที่เหมาะกับทุกคน เราจึงชอบเอกสารเชิงเทคนิคที่กระชับพร้อมการอ้างอิงถึงบทความเพิ่มเติมที่มุ่งเป้าไปยังกลุ่มผู้อ่านเฉพาะทางมากกว่า

## เอกสาร API

### เครตของ Rand

แนะนำให้ใช้ Rust เวอร์ชัน nightly เพื่อการจัดการลิงก์ที่ถูกต้อง

ในการ build เอกสาร API ทั้งหมดของทุกเครตในที่เก็บ
[rust-random/rand](https://github.com/rust-random/rand) ให้รัน:

```sh
# Optionally, enable some unstable but widely used doc features:
export RUSTDOCFLAGS="--cfg docsrs -Zunstable-options --generate-link-to-definition"

# Build doc for all crates in the workspace:
cargo doc --workspace --no-deps --all-features --open
```
(หรืออีกทางหนึ่ง ให้ดู `Cargo.toml` ภายใต้ `[package.metadata.docs.rs]` ซึ่งอาจแนะนำการตั้งค่าเฉพาะ workspace หรือเฉพาะเครต)

บน Linux การตั้งค่าให้ build ใหม่อัตโนมัติหลังการแก้ไขทุกครั้งทำได้ง่าย:
```sh
while inotifywait -r -e close_write src/ rand_*/; do cargo doc; done
```

หลังจากแก้ไขเอกสาร API เราแนะนำให้ทดสอบตัวอย่าง:

```sh
cargo test --doc
```

### เครต Getrandom

ที่เก็บ [rust-random/getrandom](https://github.com/rust-random/getrandom)
มีเพียงเครตเดียว ดังนั้นแค่ `cargo doc` ก็เพียงพอ

## เอกสารเสริม

### ไฟล์ README

ไฟล์ README บรรจุบทนำสั้นๆ เกี่ยวกับเครต ป้าย shields ลิงก์ที่มีประโยชน์ เอกสารฟีเจอร์แฟล็ก ข้อมูลสัญญาอนุญาต และอาจมีตัวอย่าง

โดยส่วนใหญ่ไฟล์เหล่านี้ไม่มีการทดสอบต่อเนื่อง
ในกรณีที่มีตัวอย่างรวมอยู่ (ปัจจุบันมีเฉพาะเครต `rand_jitter`) เราเปิดใช้การทดสอบต่อเนื่องผ่าน `doc_comment` (ดู
[lib.rs:62 เป็นต้นไป](https://github.com/rust-random/rngs/blob/master/rand_jitter/src/lib.rs#L62))

### ไฟล์ CHANGELOG

รูปแบบบันทึกการเปลี่ยนแปลงอิงตามรูปแบบ
[Keep a Changelog](http://keepachangelog.com/en/1.0.0/)

การเปลี่ยนแปลงสำคัญทั้งหมดที่ถูก merge ตั้งแต่รีลีสล่าสุดควรระบุไว้ภายใต้หัวข้อ `[Unreleased]` ที่ด้านบนของบันทึก

### หนังสือเล่มนี้

ซอร์สของหนังสือเล่มนี้อยู่ในที่เก็บ
[rust-random/book](https://github.com/rust-random/book)
มันถูก build ด้วย mdbook ซึ่งทำให้การ build และทดสอบเป็นเรื่องง่าย:

```sh
cargo install mdbook --version "^0.4"

mdbook build --open
mdbook test

# To automatically rebuild after any changes:
mdbook watch
```

โปรดทราบว่าลิงก์ในหนังสือเป็นแบบสัมพัทธ์และออกแบบให้ทำงานใน
[หนังสือที่เผยแพร่แล้ว](https://rust-random.github.io/book/) หากคุณ build หนังสือในเครื่อง คุณอาจต้องการตั้ง symbolic link ชี้ไปยัง build ของเอกสาร API ของคุณ:
```sh
ln -s ../rand/target/doc rand
```

[docs#204]: https://github.com/rust-lang/docs.rs/issues/204
