# Benchmarking Fletcher32 checksum algorithm

## Experimental setup:

- computing the checksum of a short string hard-coded in the program code:
- compute the checksum using both the VM and native code
- execute the workflow on a desktop PC and esp32 microcontroller
- run 1000 times and compute the average

Executed code:

C which gets compiled to eBPF:
```
SEC("fletcher32")
int fletcher_32(void *ctx)
{
    // Similarly to femtocontainers. The checksum algorithm is run on a 360B
    // string (i.e. it contains 360 characters.
    char *message =
        "This is a test message for the Fletcher32 checksum algorithm.\n";

    uint16_t *data = (uint16_t *)message;

    // Algorithm needs the length in words
    size_t len = strlen(message) / 2;

    uint32_t c0 = 0;
    uint32_t c1 = 0;

    // Checksum magic
    for (c0 = c1 = 0; len > 0;) {
        size_t blocklen = len;
        if (blocklen > 360 * 2) {
            blocklen = 360 * 2;
        }
        len -= blocklen;
        for (size_t i = 0; i < blocklen; i += 2) {
            char c = *(data);
            c0 = c0 + c;
            c1 = c1 + c0;
            data += 1;
        }
        c0 = c0 % 65535;
        c1 = c1 % 65535;
    }

    uint32_t checksum = (c1 << 16 | c0);

    return checksum;
}

```

Native rust:
```
fn fletcher32_native() -> u32 {
    let message = r#"This is a test message for the Fletcher32 checksum algorithm.\n"#;

    let mut length = message.len() / 2;

    let mut c0: u32 = 0;
    let mut c1: u32 = 0;

    // Checksum magic
    while length > 0 {
        let mut blocklen: usize = length;
        if blocklen > 360 * 2 {
            blocklen = 360 * 2;
        }
        length -= blocklen;
        for i in (0..blocklen).step_by(2) {
            let c;
            unsafe {
                c = *message.as_bytes().get_unchecked(i);
            }
            c0 = c0 + c as u32;
            c1 = c1 + c0;
        }
        c0 = c0 % 65535;
        c1 = c1 % 65535;
    }

    let checksum = c1 << 16 | c0;

    return checksum;
}
```
Average native: 415.00ns, Average VM: 1.49µs

## Results:

| Environment | Native   | VM      |
| ----------- | -------- | ------- |
| desktop PC  | 0.415µs  | 1.49µs  |
| esp32       | 5.22µs   | 85.23µs |


## Conclusions:

Less powerful hardware of the esp32 causes the performance penalty of running
the VM to be much larger. The overhead increased from 4x slower to almost 15x slower.
