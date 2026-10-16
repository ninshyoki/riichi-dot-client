# Riichi Mortal-S42 CPU client: recovery-v2

Download [the audited source archive](riichi-dot-client-recovery-v2-source.tar.gz) and verify it with [archive.sha256](archive.sha256).
Extract with `tar -xzf riichi-dot-client-recovery-v2-source.tar.gz`, then `cd riichi-dot-client` before following the commands below.

A four-player MJAI client using the original Mortal-S42 model, encoder and placement-aware win guard. This is a newly reconstructed source version with fresh verification. Earlier build/test results do not transfer to it. Functional testing provides correctness evidence, not a guarantee of leaderboard rank.

## Build and offline tests

Requires Linux x86_64, CPython 3.12 with venv, a C/C++ linker and development libraries. Review the script and license notices before installation. Everything installs within this checkout, using exact hash-locked binary wheels and the pinned official Rust toolchain.

```sh
python3.12 client/scripts/bootstrap.py --verify-only
python3.12 client/scripts/bootstrap.py
RAYON_NUM_THREADS=1 OMP_NUM_THREADS=1 MKL_NUM_THREADS=1 OPENBLAS_NUM_THREADS=1 .venv/bin/python -m pytest client/tests -q
.venv/bin/python -m client.offline --threads 1 --games 4 --output build/offline-report.json
```

See [BUILD.md](BUILD.md) for source identities, dependency pins and native tests. No model weights, compiled artifacts, credentials or game logs are distributed in this source tree.

## Fresh verification

134 Python tests passed in 46.86 seconds. They cover exact literal legal-action matching; all seats; malformed input; duplicate/stale request IDs; scheduling/inference/send deadlines; startup timeout; cancellation and process reaping; secure launcher behavior; 16 actual upstream all-last guard fixtures; 24 native red-call/kan/furiten cases; and two complete hanchans. One hanchan passes through four actual inference subprocesses, base64 observations, the session/connection adapter, loopback JSON transport and exact outstanding-ID assertions. Original upstream tests also passed: 28 Rust unit tests and 4 doctests.

Rare-action fixtures exercise native mechanics and action matching. They are not complete RiichiEnv-to-IPC-to-server-wire rare-position integration tests. The hanchan tests do not establish every rare interaction or playing strength.

Sequential full-path benchmark, 68 requests per configuration, on a shared 9-logical-CPU AMD EPYC 9V74 host:

| Torch threads | Startup/warmup | p50 | p95 | p99 | Maximum |
|---|---:|---:|---:|---:|---:|
| 1 | 1.24 s | 24.7 ms | 45.3 ms | 67.2 ms | 68.8 ms |
| 2 | 1.22 s | 24.0 ms | 45.4 ms | 72.0 ms | 72.4 ms |
| 4 | 1.28 s | 34.3 ms | 66.3 ms | 87.4 ms | 111.7 ms |

These measurements include observation serialization/decode, IPC, inference, legality checks and reply serialization. Network is excluded. Each configuration uses one Rayon thread and the same seed. No known heavy local build/inference job ran concurrently; shared-host scheduling was uncontrolled and CPU-pressure telemetry unavailable. Small samples are not deadline guarantees. One Torch thread is the conservative default.

## Validation launcher and current readiness

The launcher is implemented and tested with synthetic credentials and sockets. No actual token was entered and no game connection was made during this recovery.

Environment-specific blocker: the tested cloud executor currently cannot directly resolve `game.riichi.dev` (DNS failure). Do not ask an operator to enter a real token until a supported direct network route is verified. Keep the launcher's proxy, redirect and TLS safeguards intact; dependency-build proxies do not authorize proxying a bearer-token connection.

Once direct connectivity and the intended one-game submission are approved, the operator may personally run this command in a private interactive terminal:

```sh
.venv/bin/python -m client.validate_one
```

The launcher accepts no arguments. It verifies and warms the model first, then asks the user to paste and submit the token at a hidden controlling-terminal prompt. Empty Enter cancels. Only `wss://game.riichi.dev/ws/validate` is supported, with verified TLS, no redirects, no environment proxy discovery and no TLS key logging. A post-game `validation_result` with `passed: true` is required to confirm activation. The client never reconnects, queues ranked play or starts a second game.

Never put tokens in chat, command arguments, shell history, environment variables, source/config files or logs. A hidden terminal cannot protect against a compromised host; Python cannot guarantee cryptographic erasure of process memory. After unexpected disconnect, establish game-end status before another attempt.

## Protocol and integrity

- Verify the checkpoint length/SHA before restricted `torch.load(weights_only=True)`. Check exact tensor keys, shapes, dtypes, finite values and parameter count.
- Require the generated runtime source map to equal the bundled 436-file pinned map, safe source/native paths and matching hashes before importing the original runtime. No weak fallback agent.
- Use actual assigned seat, version-4 observations (1012 by 34), 46 actions, CPU FP32, epsilon zero and original `rule_based_agari`.
- Feed each `new_events()` delta once. Only the final event may invoke inference; informational socket events never double-feed state.
- Reply exactly once to the current request using a uniquely matching literal legal candidate. Preserve red fives and consumed multiplicities; add actor only if absent and permitted, and echo the exact request ID. Do not invent omitted target/tsumogiri fields.
- One subprocess owns mutable model/Bot state. Startup/action deadline, cancellation, protocol failure and disconnect invalidate the session and kill/reap it. A blocked receiver wakes on failure. No concurrent state mutations or stale replies.

## License and source

The wrapper and unchanged bundled Mortal runtime are AGPL-3.0-or-later; see [LICENSE](LICENSE) and [NOTICE](NOTICE). The separately downloaded weights are Apache-2.0. This tree includes the unchanged pinned upstream source, wrapper, tests and build instructions. Preserve applicable notices and corresponding-source availability when distributing or deploying covered software.
