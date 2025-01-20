# starskey
Starskey is a fast embedded key-value store package for GO!  Starskey implements a multi-level, durable log structured merge tree.

## Features
- **Background compaction**  Compactions occur on writes, if any disk level reaches it's max size half of the sstables are merged into a new sstable and placed into the next level.  This algorithm is recursive until last level.  At last level if full we merge all sstables into a new sstable.
- **Simple API** with Put, Get, Delete, Range, FilterKeys
- **Atomic transactions** You can group multiple operations into a single atomic transaction.  If transactions fail they rollback.
- **Configurable options** You can configure many options such as max levels, memtable threshold, bloom filter, and more.
- **WAL with recovery** Starskey uses a write ahead log to ensure durability.  Memtable is replayed if a flush did not occur prior to shutdown.  On sorted runs to disk the WAL is truncated.
- **Key value separation** Keys and values are stored separately for sstables.
- **Bloom filters** Each sstable has an in memory bloom filter to reduce disk reads.
- **Fast** up to 400k+ ops per second.
- **Compression** Snappy compression is available.
- **Logging** Logging to file is available.
- **Thread safe** Starskey is thread safe.

## Basic Example
Below is a basic example of how to use starskey.
```go
import (
    "github.com/starskey-io/starskey"
)


// Test opening starskey with a valid configuration
starskey, err := Open(&Config{
        Permission:     0755,           // Dir, file permission
        Directory:      "test",         // Directory to store data
        FlushThreshold: 1024 * 1024,    // Flush threshold in bytes
        MaxLevel:       3,              // Max levels number of disk levels
        SizeFactor:     10,             // Size factor for each level.  Say 10 that's 10 * the FlushThreshold at each level. So level 1 is 10MB, level 2 is 100MB, level 3 is 1GB.
        BloomFilter:    false,          // If you want to use bloom filters
        Logging:        true,           // Enable logging to file
        Compression:    false,          // Enable compression (Snappy)
    })                                  // Config cannot be nil**
if err != nil {
    t.Fatalf("Failed to open starskey: %v", err)
}

// Close starskey
if err := starskey.Close(); err != nil {
    t.Fatalf("Failed to close starskey: %v", err)
}

key := []byte("some_key")
value := []byte("some_value")

if err := starskey.Put(key, value); err != nil {
    t.Fatalf("Failed to put key-value pair: %v", err)
}

v, err := starskey.Get(key)
if err != nil {
    t.Fatalf("Failed to get key-value pair: %v", err)
}

// Value is nil if key does not exist
if v == nil {
    t.Fatalf("Value is nil")
}

fmt.Println(string(key), string(v))
```

## Range
Using range method to get a range of values between two keys.
```go
results, err := starskey.Range([]byte("key900"), []byte("key980"))
if err != nil {
    t.Fatalf("Failed to range: %v", err)
}
```

## FilterKeys
Using filter keys to filter keys based on a function.
```go
compareFunc := func(key []byte) bool {
    // if has prefix "c" return true
    return bytes.HasPrefix(key, []byte("c"))
}

results, err := starskey.FilterKeys(compareFunc)
if err != nil {
    t.Fatalf("Failed to filter: %v", err)
}
```


## Atomic Transactions
Using atomic transactions to group multiple operations into a single atomic transaction.
```go
txn := starskey.BeginTxn()
if txn == nil {
    t.Fatalf("Failed to begin transaction")
}

txn.Put([]byte("key"), []byte("value"))
// or txn.Delete([]byte("key"))

if err := txn.Commit(); err != nil {
    t.Fatalf("Failed to commit transaction: %v", err)
}
```

## Delete
Delete a key from starskey.
```go
if err := starskey.Delete([]byte("key")); err != nil {
    t.Fatalf("Failed to delete key: %v", err)
}
```