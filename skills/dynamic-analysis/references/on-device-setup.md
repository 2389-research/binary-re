# On-Device Analysis

**When emulation fails or device-specific behavior needed.**

## Remote GDB via gdbserver

```bash
# On target device (via SSH/ADB)
gdbserver :1234 ./binary

# On host (with port forward)
ssh -L 1234:localhost:1234 user@device &
gdb-multiarch -q \
  -ex "target remote localhost:1234" \
  ./binary
```

## Remote strace (if available)

```bash
# On target device
strace -f -o /tmp/trace.log ./binary

# Pull log
scp user@device:/tmp/trace.log .
```
