# Vendored rocm-xio headers

Files in this directory are byte-identical snapshots of upstream
[rocm-xio](https://github.com/ROCm/rocm-xio) headers. They are vendored so
TransferBench builds without needing the rocm-xio source tree on the host;
the matching `librocm-xio.so` is resolved at runtime via `dlopen` in
[`../xiodl.hpp`](../xiodl.hpp).

## File map

| Vendored path | Upstream path |
|---|---|
| `sdma-ep.h` | `rocm-xio/src/endpoints/sdma-ep/sdma-ep.h` |
| `sdma_pkt_struct.h` | `rocm-xio/src/endpoints/sdma-ep/sdma_pkt_struct.h` |
| `sdma_opcodes.h` | `rocm-xio/src/endpoints/sdma-ep/sdma_opcodes.h` |
| `xio-export.h` | `rocm-xio/src/include/xio-export.h` |

License: MIT (see SPDX line at the top of each file).

## Re-syncing on a rocm-xio bump

```bash
ROCM_XIO=/path/to/rocm-xio
cp "$ROCM_XIO/src/endpoints/sdma-ep/sdma-ep.h"        sdma-ep.h
cp "$ROCM_XIO/src/endpoints/sdma-ep/sdma_pkt_struct.h" sdma_pkt_struct.h
cp "$ROCM_XIO/src/endpoints/sdma-ep/sdma_opcodes.h"   sdma_opcodes.h
cp "$ROCM_XIO/src/include/xio-export.h"               xio-export.h
```

## ABI contract reminder

`SdmaQueueHandle`'s field layout and the SDMA packet bitfields in
`sdma_pkt_struct.h` are part of an ABI shared with `librocm-xio.so`. If a
re-synced version of these headers ever appears alongside an older
`librocm-xio.so` (or vice versa), kernels will silently encode wrong
packets. Re-sync the headers and rebuild rocm-xio together.

## Why these specific files

- `sdma-ep.h` declares the public API and defines the `__device__
  __forceinline__` templates (`putSignal`, `waitSignal`, `SdmaQueueHandle`
  methods) that must be inlined into TransferBench's kernel.
- `sdma_pkt_struct.h` and `sdma_opcodes.h` are transitive deps of
  `sdma-ep.h`.
- `xio-export.h` is included by the patched `sdma-ep.h` to pick up the
  `XIO_API` no-op macro on the consumer side.

We deliberately do **not** vendor `xio.h` itself; the two host symbols we
need from it (`xio::allocDeviceMemory`, `xio::freeDeviceMemory`) are
declared directly in `xiodl.hpp` to avoid pulling in `xio.h`'s five
transitive headers.
