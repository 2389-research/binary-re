# Frida Setup and Function Hooking

**Intercept function calls without modifying binary.**

⚠️ **Architecture Constraint:** Frida requires native-arch execution. It **cannot** attach to QEMU-user targets.

| Scenario | Works? | Alternative |
|----------|--------|-------------|
| Native binary (x86_64 on x86_64) | ✅ | - |
| Cross-arch under QEMU-user | ❌ | Use on-device frida-server |
| Docker native-arch container | ✅ | - |
| Docker cross-arch (emulated) | ❌ | Use on-device frida-server |

For cross-arch Frida, deploy `frida-server` to the target device:
```bash
# On target device:
./frida-server &

# On host:
frida -H device:27042 -f ./binary -l hook.js --no-pause
```

## Basic Hook

```javascript
// hook_connect.js
Interceptor.attach(Module.findExportByName(null, "connect"), {
  onEnter: function(args) {
    console.log("[connect] Called");
    var sockaddr = args[1];
    var family = sockaddr.readU16();
    if (family == 2) { // AF_INET
      var port = sockaddr.add(2).readU16();
      var ip = sockaddr.add(4).readByteArray(4);
      console.log("  Port: " + ((port >> 8) | ((port & 0xff) << 8)));
      console.log("  IP: " + new Uint8Array(ip).join("."));
    }
  },
  onLeave: function(retval) {
    console.log("  Return: " + retval);
  }
});
```

```bash
# Run with Frida
frida -f ./binary -l hook_connect.js --no-pause
```

## Tracing All Calls to Library

```javascript
// trace_libcurl.js
var libcurl = Process.findModuleByName("libcurl.so.4");
if (libcurl) {
  libcurl.enumerateExports().forEach(function(exp) {
    if (exp.type === "function") {
      Interceptor.attach(exp.address, {
        onEnter: function(args) {
          console.log("[" + exp.name + "] called");
        }
      });
    }
  });
}
```

## Memory Inspection

```javascript
// dump_memory.js
var base = Module.findBaseAddress("binary");
console.log("Base: " + base);

// Dump region
var data = base.add(0x1000).readByteArray(256);
console.log(hexdump(data, { offset: 0, length: 256 }));
```
