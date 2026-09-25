# Docker-Based Cross-Architecture Analysis (macOS)

**Use Docker for cross-arch execution when native QEMU unavailable.**

## ARM32 Binary on macOS

```bash
docker run --rm --platform linux/arm/v7 \
  -v ~/code/samples:/work:ro \
  arm32v7/debian:bullseye-slim \
  sh -c '
    # Fix linker path mismatch (common issue)
    ln -sf /lib/ld-linux-armhf.so.3 /lib/ld-linux.so.3 2>/dev/null || true

    # Install dependencies if needed (check rabin2 -l output)
    apt-get update -qq && apt-get install -qq -y libcap2 libacl1 2>/dev/null

    # Run with library debug output (strace alternative)
    LD_DEBUG=libs /work/binary args
  '
```

## ARM64 Binary on macOS

```bash
docker run --rm --platform linux/arm64 \
  -v ~/code/samples:/work:ro \
  arm64v8/debian:bullseye-slim \
  sh -c 'LD_DEBUG=libs /work/binary args'
```

## x86 32-bit Binary on macOS

```bash
docker run --rm --platform linux/i386 \
  -v ~/code/samples:/work:ro \
  i386/debian:bullseye-slim \
  sh -c '/work/binary args'
```

## Tracing Limitations in Docker/QEMU User-Mode

| Method | Works? | Alternative |
|--------|--------|-------------|
| strace | ❌ (ptrace not implemented) | `LD_DEBUG=files,libs` |
| ltrace | ❌ (same reason) | Direct observation or Frida |
| gdb | ✓ (with QEMU `-g` flag) | N/A |

## LD_DEBUG Options (strace alternative)

```bash
LD_DEBUG=libs     # Library search and loading
LD_DEBUG=files    # File operations during loading
LD_DEBUG=symbols  # Symbol resolution
LD_DEBUG=bindings # Symbol binding details
LD_DEBUG=all      # Everything (verbose)
```
