Stage: Finished


# Hashing

SSZ utilizes the SHA-256 hash function.

The standard specification for SHA-256 can be found in [FIPS 180-4](https://csrc.nist.gov/publications/detail/fips/180/4/final).

## Hashing primitive for binary trees.

```
H(a: bytes32, b: bytes32) -> SHA_256(a ++ b)
```

Where `++` is concatenation, i.e. tightly packing `a` and `b` into 64 bytes.
And `SHA_256` is run on the standard unmodified pre-state, and returns the digest after writing and processing the above 64 bytes.

## Zero-hashes

A common occurrence in merkleization in SSZ is `H(X, H(0, 0)), H(X, H(H(0, 0), H(0, 0))), H(X, H(H(H(...`

The right-hand side here would be costly to merkleize leaf by leaf, but is efficiently precomputed, and referred to as a "zero hash" of some order N, starting from 0 being a bare zeroed `bytes32`:

```
Z[0]: 000000....   # a zeroed bytes32
Z[1]: H(Z[0], Z[0])
Z[2]: H(Z[1], Z[1])
Z[2]: H(Z[1], Z[1])
Z[3]: H(Z[2], Z[2])
...
```

## Alternatives

Other hash-functions have been considered for specific use cases, but are actively avoided for higher compatibility with other platforms, and more consistency.

- 256 bits SHA-3 and Keccak were considered but dropped in favor of SHA-256 support and compatibility.
- "fast-SHA256" as described in [BIP-98](https://github.com/bitcoin/bips/blob/master/bip-0098.mediawiki), optimized for two 32 byte inputs. But still a draft, and even more compatibility concerns.
- A S\[T/N]ARK-friendly hash function, the aim is to migrate in a future Eth2 deployment phase.
