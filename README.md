# ptau

Powers of Tau ceremony files for the Lelantos circuits, served as release assets.

These are public, immutable build inputs — not secrets. They live here because the
upstream host they used to be fetched from, `https://storage.googleapis.com/zkevm/ptau`,
started returning `403 AccessDenied` for the whole bucket, and every other known public
mirror (`hermez.s3-eu-west-1.amazonaws.com`, `pse-trusted-setup-ppot.s3.eu-central-1.amazonaws.com`)
is dead too. A build that cannot fetch its ptau cannot run a phase-2 setup.

## What a ptau file is

Groth16 needs a structured reference string, produced in two phases. Phase 1 is
universal — independent of any particular circuit — and is what a `.ptau` file holds:
encrypted powers of a secret τ in both curve groups (`[τ⁰]G₁, [τ¹]G₁, …, [τⁿ]G₁`, the
G₂ counterparts, and the alpha/beta variants). Phase 2, run per circuit by
`snarkjs groth16 setup`, specialises that SRS into a proving and verifying key.

τ itself is never known to anyone: the file is the output of a multi-party ceremony
where each contributor mixes in their own randomness and destroys it. Soundness holds
as long as at least one contributor was honest. If every contributor colluded, they
could forge proofs that verify against keys derived from this file.

These are the **Hermez** ceremony files (`powersOfTau28_hez_final_*`) over BN254 — the
same artifacts snarkjs has always distributed, republished here byte-for-byte.

## Files

Power `N` supports `2^N` constraints, where snarkjs sizes the domain from
`nConstraints + nPubInputs + nOutputs`.

| File | Power | Max constraints | Size | SHA-256 |
| --- | --- | --- | --- | --- |
| `powersOfTau28_hez_final_16.ptau` | 16 | 65,536 | 72 MiB | `1c401abb57c9ce531370f3015c3e75c0892e0f32b8b1e94ace0f6682d9695922` |
| `powersOfTau28_hez_final_17.ptau` | 17 | 131,072 | 144 MiB | `6b662a324867139fb1a20a324d90b6ff61856dfb23f59326909f14b0e2483ae0` |
| `powersOfTau28_hez_final_20.ptau` | 20 | 1,048,576 | 1.1 GiB | `159d3f938d941e06767d99f30b9fe59a245400a4aae138cf8e411732d7a2f6cd` |

`lelantos-org/circuits` builds on the 2^17 file: Transact(11,4,6) is 100,320
constraints and TreeUpdateBatch(11,8) is 113,502, so neither fits 2^16.

## Fetching

Release assets are unauthenticated and unmetered — plain `curl`, no token, no LFS
bandwidth quota:

```sh
BASE=https://github.com/lelantos-org/ptau/releases/download/hermez
curl -fL --retry 3 "$BASE/powersOfTau28_hez_final_17.ptau" -o powersOfTau28_hez_final_17.ptau
```

`-f` matters: without it curl happily writes an HTTP error page into the output file,
and the failure only surfaces later as `snarkjs: Error: … Invalid File format`.

Always check the digest before use — a ptau is a security-relevant input:

```sh
echo "6b662a324867139fb1a20a324d90b6ff61856dfb23f59326909f14b0e2483ae0  powersOfTau28_hez_final_17.ptau" | sha256sum -c
```

To verify the ceremony transcript itself rather than just the bytes (slow — tens of
minutes, and it re-checks every contribution):

```sh
npx snarkjs powersoftau verify powersOfTau28_hez_final_17.ptau
```

## Provenance

Redistributed from the Hermez phase-1 ceremony, unmodified. This repo adds no
contribution of its own and makes no claim about the ceremony's security beyond what
the original participants established.
