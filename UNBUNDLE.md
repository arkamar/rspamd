# Rspamd unbundle

This is an ongoing effort to unbundle libraries from rspamd.

## Table

| unb       | pat     | In gentoo                   | Version | license       | package                                             | Notes |
|-----------|---------|-----------------------------|---------|---------------|-----------------------------------------------------|-------|
| No        | ??      |                             |         | Apache-2.0    | [google-ced](https://github.com/google/compact_enc_det) | No release |
| No        | Yes     |                             |         | BSD-2         | [libucl](https://github.com/vstakhov/libucl)        | His own library |
| No        | Yes     |                             |         | BSD-2         | [librdns](https://github.com/vstakhov/librdns)      | His own library. No release |
| No        | Yes     |                             |         | public-dom,CC0| [libottery](https://github.com/nmathewson/libottery)| Heavily patched. Upstream seems to be dead |
| No        | Yes     |                             |         | MIT           | [kann](https://github.com/attractivechaos/kann)     | No release yet, Upstream does not create shared library |
| No        | Yes     |                             | ?       | MIT           | [mumhash](https://github.com/vnmakarov/mum-hash)    | Few changes but it seems to be compatible |
| No        | Yes     |                             |         | BSL-1.0       | [fpconv](https://github.com/night-shift/fpconv)     | |
| No        | Yes     |                             | ?       | BSD           | lc-btrie                                            | I was not able to locate any sort of original source |
| No        | Yes     |                             |         | LGPL-3.0      | [aho-corasick](https://github.com/mischasan/aho-corasick) | No release in upstream |
| No        | ?(Yes)  |                             | ?       | MIT           | fastutf8                                            | Unable to locate upstream |
| No        | No      |                             | 0.0.2   | BSD, Unicode  | [replxx](https://github.com/AmokHuginnsson/replxx)  | |
| No        | ??      |                             | ??      | zlib          | [t1ha](https://github.com/erthink/t1ha)             | |
| No        | No      |                             | 2.0.?+  | MIT           | [lua-tableshape](https://github.com/leafo/tableshape)| In middle between 2.0.0 and 2.1.0 |
| No        | No      |                             | 1.0     | MIT           | [lua-lupa](https://foicica.com/lupa/)               | |
| No        | No      |                             | 0.1.3+  | MIT           | [lua-fun](https://github.com/luafun/luafun)         | Upstream seems to be dead. Bundled version is somewhere in between 0.1.3 and master |
| No        | Notes   | `dev-lua/lua-argparse`      |         |               | lua-argparse                                        | This package has upstream version [argparse-0.6.0](https://github.com/mpeterv/argparse/releases) released in April 2018, which is in `::gentoo`. However rspamd uses version [0.7.0](https://github.com/pauloue/argparse/releases) released by Paul Ouellette in August 2019. |
| No        | Yes     | `net-libs/http-parser`      | 2.2.0   | MIT           | http-parser                                         | Added some SPAMC compatibility http methods |
| No        | Yes     | `dev-libs/uthash`           | 1.9.8   | BSD-1         | uthash                                              | Added `LL_REVERSE` and `LL_REVERSE2` macros. It looks doable |
| No        | Yes     | `dev-libs/libev`            | 4.33    | BSD           | libev                                               | **Candidate**; see notes below |
| No        | Yes     | `dev-db/tinycdb`            | 0.77    | public-domain | cdb                                                 | Seems to be heavily patched |
| No        | Yes     | `dev-libs/xxhash`           |         | BSD-2         | xxhash                                              | Bundled library is modified with `XXH32_init` function and extra `XXH32_freeState` calls, but it seems that it is not used anymore. It is possible to unbundle it with defining `XXH_STATIC_LINKING_ONLY` before `#include "xxhash.h"` in `src/libcryptobox/cryptobox.c`, which allows compiler to measure the struct size. |
| **Yes**   | Yes     | `dev-libs/hiredis`          |         | BSD           | hiredis                                             | https://github.com/rspamd/rspamd/pull/3313 |
| **Yes**   | No      | `dev-lua/LuaBitOp`          |         | MIT           | lua-bit                                             | |
| **Yes**   | No      | `dev-lua/lpeg`              |         | MIT           | lua-lpeg                                            | |
| **Yes**   | No      | `dev-libs/snowball-stemmer` |         | BSD           | snowball                                            | |
| **Yes**   | No      | `app-arch/zstd`             |         | BSD           | zstd                                                | |

## The rest

| Name                                     | Description |
|------------------------------------------|-------------|
| [elastic](contrib/elastic)               | Some json data |
| [exim](contrib/exim)                     | Some patches, they are not used in rspamd directly |
| [publicsuffix](contrib/publicsuffix)     | tld list with some perl script |
| [languages-data](contrib/languages-data) | some language data |

## Notes

* [`contrib/lua-bit`](contrib/lua-bit) - This package corresponds to `dev-lua/LuaBitOp` package in `::gentoo`. It is called directly in `src/lua/lua_common.c`. It is used directly in function `rspamd_lua_init` where it is passed to `rspamd_lua_add_preload` function, which seems to be function for modification of global table? The same happens with `lpeg` and `ucl`. It seems to be save to just ignore this call.
* [`contrib/libev`](contrib/libev) - This one is also modified. It introduces
```c
#define ev_can_stop(ev)                      (ev_is_pending(ev) || ev_is_active(ev)) /* ro, true when the watcher has been started */
```
and
```c
EV_API_DECL void ev_now_update_if_cheap (EV_P) EV_NOEXCEPT;
EV_API_DECL int ev_active_cnt (EV_P) EV_NOEXCEPT;
```
The first one is simple as it could be defined in the project, and the second could be substituted with `ev_now_update` and `ev_active_cnt` is used only in `rspamd_worker_check_finished` which is not used at all.
