# A simple diagram on how db storage engine works

- 

```text
                        CLIENT APPLICATION
                get() / put() / delete() / scan()
                                │
                                ▼
  ┌────────────────────────────────────────────────────────────┐
  │ [6] TRANSACTION MANAGER                                    │
  │  • Lock manager (2PL)  or  MVCC snapshots                  │
  │  • begin / commit / abort, deadlock detection              │
  └────────────────────────────────────────────────────────────┘
                               ▲ ▼
  ┌────────────────────────────────────────────────────────────┐
  │ [4] INDEX  (B+Tree  or  LSM-tree)                          │
  │  • Maps key -> (page_id, slot); keeps keys sorted          │
  │  • get / put / delete / range scan                         │
  └────────────────────────────────────────────────────────────┘
                               ▲ ▼
  ┌────────────────────────────────────────────────────────────┐
  │ [2] PAGE LAYER (on-disk layout)                            │
  │  • Fixed-size pages (4/8/16 KB), slotted records           │
  │  • Page header: page_id, LSN, checksum                     │
  └────────────────────────────────────────────────────────────┘
                               ▲ ▼
  ┌────────────────────────────────────────────────────────────┐
  │ [3] BUFFER POOL MANAGER                                    │
  │  • Page cache: frames, pin counts, dirty bits              │
  │  • Eviction: LRU / Clock / LRU-K                           │
  └────────────────────────────────────────────────────────────┘
                               ▲ ▼
  ┌────────────────────────────────────────────────────────────┐
  │ [5] WAL + RECOVERY MANAGER                                 │
  │  • Log every change BEFORE page hits disk                  │
  │  • Checkpoints, crash recovery (ARIES)                     │
  └────────────────────────────────────────────────────────────┘
                               ▲ ▼
  ┌────────────────────────────────────────────────────────────┐
  │ [1] DISK I/O LAYER                                         │
  │  • pread / pwrite (aligned), fsync, O_DIRECT               │
  └────────────────────────────────────────────────────────────┘
                 ▼                             ▼
          ┌──────────────┐              ┌──────────────┐
          │   data.db    │              │   wal.log    │
          │ (pages 8 KB) │              │ (append-only)│
          └──────────────┘              └──────────────┘
```
