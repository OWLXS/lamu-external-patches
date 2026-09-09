# lamu-external-patches

Patches applied to the lamu kernel (Motorola moto g05/g15/g15 power) that
**have no dedicated fork to host the commit** -- `kernel/common` (GKI) is
consumed directly from Google's Gerrit, so any change to it is kept here
for reference and reapplication.

Two categories: third-party patches (credited to their original authors)
and our own lamu-specific fixes.

## Third-party patches

Curated from [ahmed-alnassif/GKID-Kernels](https://github.com/ahmed-alnassif/GKID-Kernels)
(`kernel-patches/common/`), which itself collects patches from several
sources.

### `0018-sched-core-Adjusting-the-order-of-scanning-CPU.patch`
Fixes `select_idle_capacity()`/`task_numa_assign()` to start scanning for
an idle CPU from the **next** CPU instead of re-checking the target CPU
that was already checked. Small scheduler efficiency fix.

**Real Linux mainline commit.** Original author: Hao Jia
(`jiahao.os@bytedance.com`). Signed-off-by: Peter Zijlstra (Intel).
Reviewed-by: Vincent Guittot. Acked-by: Mel Gorman.
Link: https://lore.kernel.org/r/20221216062406.7812-3-jiahao.os@bytedance.com

### `0020-sched-fair-Disable-CACHE_HOT_BUDDY-to-leverage-Dynam.patch`
Disables the scheduler's `CACHE_HOT_BUDDY` feature -- on SoCs with a
**DynamIQ Shared Unit** (like the lamu's MT6768), L2/L3 cache locality
isn't lost when migrating a task between cores in the same cluster, so
this heuristic (designed for topologies without shared cache) hurts more
than it helps.

Author: Sultan Alsawaf (`sultan@kerneltoast.com`).

### `0022-net-sock-increase-default-number-of-_SK_MEM_PACKETS-.patch`
Increases `_SK_MEM_PACKETS` from 256 to 1024 (used in the
`SK_WMEM_MAX`/`SK_RMEM_MAX` calculation) -- larger socket buffers, better
network throughput at the cost of more RAM per connection.

Author: Colin Ian King (`colin.i.king@intel.com`).

## Our own fixes (lamu)

### `lamu-0001-arm64-gki_defconfig-disable-CFI_CLANG.patch`
Disables `CONFIG_CFI_CLANG` in `gki_defconfig`.

The module tree (`kernel_device_modules-6.6`) already disabled CFI via
`lamu_overlay.config`, but the GKI kernel kept it enabled. A CFI-enabled
kernel only accepts indirect calls into a module carrying the matching
type id -- so first-stage init died on the **first** module it tried to
load:

```
init: Loading module /lib/modules/bootprof.ko with args ''
CFI failure at do_one_initcall+0xd4/0x360
        (target: init_module+0x0/0xf7c [bootprof]; expected type: 0x36b1c5a6)
Kernel panic - not syncing: Oops - CFI: Fatal exception
```

Captured from `expdb` on real hardware (MT6769V/CB). The kernel boots
normally up to `Run /init as init process`; only the handoff to modules
breaks.

Disabling on the kernel side is the permissive direction: a kernel
without CFI loads modules with or without CFI, the reverse doesn't hold.
This also matters for the 434 prebuilt vendor blobs, which are built
against Motorola's own kernel and can't be recompiled.

**CFI needs to be consistent across both layers** -- touching only the
module overlay isn't enough.

## Applying

```bash
cd kernel-6.6/  # or wherever the local kernel/common is
git apply /path/to/0018-....patch
git apply /path/to/0020-....patch
git apply /path/to/0022-....patch
git apply /path/to/lamu-0001-....patch
```

The three third-party patches have already been checked for ABI/CRC
impact on lamu's proprietary vendor modules -- none touch shared struct
layout (only scheduler logic or a `#define` constant), safe per the rule
documented in the main project. `lamu-0001` only touches `defconfig`, no
CRC impact.
