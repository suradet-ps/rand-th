# การทดสอบฟังก์ชันที่ใช้ RNG

บางครั้งฟังก์ชันที่ใช้ตัวสร้างเลขสุ่มอาจจำเป็นต้องได้รับการทดสอบ สำหรับฟังก์ชันที่ต้องทดสอบด้วยเวกเตอร์ทดสอบ อาจใช้วิธีต่อไปนี้:

```rust
use rand::{TryCryptoRng, rngs::SysRng};

pub struct CryptoOperations<R: TryCryptoRng = SysRng> {
    rng: R
}

impl<R: TryCryptoRng> CryptoOperations<R> {
    #[must_use]
    pub fn new(rng: R) -> Self {
        Self {
            rng
        }
    }

    pub fn xor_with_random_bytes(&mut self, secret: &mut [u8; 8]) -> [u8; 8] {
        let mut mask = [0u8; 8];
        self.rng.try_fill_bytes(&mut mask).unwrap();

        for (byte, mask_byte) in secret.iter_mut().zip(mask.iter()) {
            *byte ^= mask_byte;
        }

        mask
    }
}

fn main() {
    let rng = SysRng;
    let mut crypto_ops = <CryptoOperations>::new(rng);

    let mut secret: [u8; 8] = *b"\x00\x01\x02\x03\x04\x05\x06\x07";
    let mask = crypto_ops.xor_with_random_bytes(&mut secret);
    
    println!("Modified Secret (XORed): {:?}", secret);
    println!("Mask: {:?}", mask);
}
```

ในการทดสอบสิ่งนี้ เราสร้าง `MockCryptoRng` ที่ implement `TryRngCore` และ `TryCryptoRng` ในโมดูลทดสอบของเราได้ โปรดทราบว่า `MockCryptoRng` เป็น private และ `#[cfg(test)] mod tests` ถูกควบคุมด้วย cfg เฉพาะสภาพแวดล้อมทดสอบของเรา จึงมั่นใจได้ว่า `MockCryptoRng` จะไม่ถูกนำไปใช้ในโปรดักชันโดยไม่ตั้งใจ

```rust,noplayground
#[cfg(test)]
mod tests {
    use super::*;

    #[derive(Clone, Copy, Debug)]
    struct MockCryptoRng {
        data: [u8; 8],
        index: usize,
    }

    impl MockCryptoRng {
        fn new(data: [u8; 8]) -> MockCryptoRng {
            MockCryptoRng {
                data,
                index: 0,
            }
        }
    }

    impl CryptoRng for MockCryptoRng {}

    impl RngCore for MockCryptoRng {
        fn next_u32(&mut self) -> u32 {
            unimplemented!()
        }

        fn next_u64(&mut self) -> u64 {
            unimplemented!()
        }

        fn fill_bytes(&mut self, dest: &mut [u8]) {
            for byte in dest.iter_mut() {
                *byte = self.data[self.index];
                self.index = (self.index + 1) % self.data.len();
            }
        }

        fn try_fill_bytes(&mut self, dest: &mut [u8]) -> Result<(), rand::Error> {
            unimplemented!()
        }
    }

    #[test]
    fn test_xor_with_mock_rng() {
        let mock_crypto_rng = MockCryptoRng::new(*b"\x57\x88\x1e\xed\x1c\x72\x01\xd8");
        let mut crypto_ops = CryptoOperations::new(mock_crypto_rng);

        let mut secret: [u8; 8] = *b"\x00\x01\x02\x03\x04\x05\x06\x07";
        let mask = crypto_ops.xor_with_random_bytes(&mut secret);
        let expected_mask = *b"\x57\x88\x1e\xed\x1c\x72\x01\xd8";
        let expected_xored_secret = *b"\x57\x89\x1c\xee\x18\x77\x07\xdf";

        assert_eq!(secret, expected_xored_secret);
        assert_eq!(mask, expected_mask);
    }
}
```
