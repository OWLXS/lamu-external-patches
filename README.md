# lamu-external-patches

Third-party patches applied to the lamu kernel (Motorola moto g05/g15/g15
power) that **have no dedicated fork to host the commit** -- kept here
for reference and reapplication, credited to their original authors.

Curated from [ahmed-alnassif/GKID-Kernels](https://github.com/ahmed-alnassif/GKID-Kernels)
(`kernel-patches/common/`), which itself collects patches from several
sources.

## Patches

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

## Applying

```bash
cd kernel-6.6/  # or wherever the local kernel/common is
git apply /path/to/0018-....patch
git apply /path/to/0020-....patch
git apply /path/to/0022-....patch
```

All three have already been checked for ABI/CRC impact on lamu's
proprietary vendor modules -- none touch shared struct layout (only
scheduler logic or a `#define` constant), safe per the rule documented
in the main project.
