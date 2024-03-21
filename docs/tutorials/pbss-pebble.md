---
sidebar_label: Run with PBSS and PebbleDB
description: Introduce to running opBNB Node with PBSS and Pebble
---

# Run with PBSS(Path-Base Scheme Storage) and Pebble DB

## How to run op-geth with PBSS and Pebble DB

Add flags when starting op-geth.

```bash
--state.scheme path --db.engine pebble
```

The log outputs information indicating that PBSS and Pebble DB have been started successfully.

```bash
INFO [03-21|07:00:25.684] Using pebble as the backing database
INFO [03-21|07:00:47.039] State scheme set by user                 scheme=path
```

## PBSS(Path-Base Scheme Storage)

In the PBSS, the trie nodes are saved in disk with the encoded path and specific key prefix as key.
So the PBSS's MPT will override the older one because of the same key for account trie and storage 
trie, it can not only achieve the goal of **pruning online** but also can **reduce data redundancy dramatically**.

PBSS consists of 128 difflayers (in memory) and one disklayer, which can be illustrated by the following 
diagram. Difflayer only stores changed state data.

```bash
+-------------------------------------------+
| Block X+128 State                         |
+-------------------------------------------+
| Block X+127 State                         |
+-------------------------------------------+
|              .......                      |
+-------------------------------------------+
| Block X+1 State,  Bottom-most diff layer  |
+-------------------------------------------+
| Block X State, Disk layer(singleton trie) |
+-------------------------------------------+
```

**PBSS has better read performance**, trie access and iteration is much faster than before, store 
a single version of the state trie persisted to disk, and keep new tries (state/storage/account 
trie changes) in memory only.

### Restriction

* **Only supports querying the status data of the last 129 blocks**
  
  The RPC requests that require lookup longer status data return the error `missing trie node ${hash} (path ${path})`.

* **The withdrawal function of opBNB is not supported**
  
  It may query the status data of one hour before to get withdrawal proof.
  
  > Later version will support withdrawal.


## PebbleDB

PebbleDB was adopted in go-ethereum as well, which has become the default database for the community.

opBNB PBSS replaces LevelDB with the optimized Pebble DB. LevelDB operates without a throttle mechanism 
for flushes and compactions, consistently running at maximum speed and leading to notable latency spikes 
for both write and read operations.

On the other hand, PebbleDB employs separate rate limiters for flushes and compactions. This mechanism 
ensures operations occur only as fast as necessary, **preventing unnecessary strain on disk bandwidth**.

## FAQ

