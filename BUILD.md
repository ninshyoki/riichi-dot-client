# Pinned CPU build

The executable recipe is `client/scripts/bootstrap.py`. It verifies all bundled upstream source hashes before building, downloads checksum-pinned weights and official installer data, installs exact hash-locked binary Python wheels, preserves Cargo.lock and generates a locally bound native-library hash manifest. No model-provided loader or downloaded shell script is executed. The official rustup executable is verified before execution.

Prerequisites: Linux x86_64, CPython 3.12 with venv, C/C++ linker/development libraries and access to official HTTPS registries. Tested Python is 3.12.14. Initial setup needs internet access; tests are offline. Standard platform HTTP(S)/ALL/NO_PROXY variables may be inherited only for checksum-locked dependency acquisition/build, without printing or saving their values. Rust distribution, Cargo registry and compiler override variables are not inherited. Credential-bearing validation separately fixes proxy=None.

- Runtime: https://github.com/Equim-chan/Mortal/tree/0cff2b52982be5b1163aa9a62fb01f03ce91e0d2 ; all 436 tracked files are bundled unchanged, without .git. Hashes are in `client/upstream-source-manifest.json`.
- Original Cargo.lock Git blob: `be34fed862bccdfd621e90c6ded0a1ee640f7ee9`; SHA-256: `5f7a562e3c428b0ae7f078db63231c201a25f6ff1be16fd5e7b37df3b226dcb2`. Dependencies use crates.io registry checksums. Original Cargo/package metadata identifies their source repositories and licenses.
- Rust: 1.88.0. Rustup installer 1.28.2 from https://static.rust-lang.org/rustup/archive/1.28.2/x86_64-unknown-linux-gnu/rustup-init ; SHA-256 `20a06e644b0d9bd2fbdbfd52d42540bdde820ea7df86e92e533c073da0cdd43c`.
- Python wheels: PyTorch 2.13.0+cpu from https://download.pytorch.org/whl/cpu ; NumPy 2.3.3, RiichiEnv 0.4.10, websockets 15.0.1, pytest and supporting packages from https://pypi.org/simple . Exact wheel identities and SHA-256 values are in `client/requirements-hashed.lock` and `client/wheels.sha256`. Package metadata provides dependency source links and licenses. The wheel lock is CPython 3.12/Linux x86_64 specific.
- Weights: https://huggingface.co/haor/Mortal-S42/tree/524981f21a929e3c3b269f65af79b84163bd02ca ; `weights/s42-inference.pth`, 43,670,035 bytes, SHA-256 `e5f6955b6f19007fd65315e53291b36994493046bc118b991b959e9dc596d660`. Apache-2.0; downloaded separately, not included here.

All generated/install files are confined to `.venv`, `build` and `assets/Mortal-S42/weights`. Rust uses `--no-modify-path`. No global installation, native-CPU optimization flag or security-setting change is required. Build only the extension with `cargo build --release --locked -p libriichi --lib -j4`; unrelated executable targets have different link requirements. Source/dependency identities are pinned; binary hashes can vary across hosts/linkers and are freshly generated locally.

Original Rust tests:

```sh
PYLIB=$(.venv/bin/python -c 'import sys;print(sys.base_prefix+"/lib")')
CARGO_HOME="$PWD/build/cargo-home" RUSTUP_HOME="$PWD/build/rustup-home" CARGO_TARGET_DIR="$PWD/build/mortal-tests" PYO3_PYTHON="$PWD/.venv/bin/python" RUSTFLAGS="-L native=$PYLIB" RUSTDOCFLAGS="-L native=$PYLIB" LD_LIBRARY_PATH="$PYLIB" build/cargo-home/bin/cargo +1.88.0 test --manifest-path vendor/Mortal-upstream/Cargo.toml --workspace --locked --no-default-features --features flate2/zlib -j4
```

Sequential full-path benchmarks, after builds and other inference finish:

```sh
.venv/bin/python -m client.benchmark_ipc --threads 1
.venv/bin/python -m client.benchmark_ipc --threads 2
.venv/bin/python -m client.benchmark_ipc --threads 4
```

Official protocol references: https://riichi.dev/docs/protocol , https://riichi.dev/docs/mortal , https://riichi.dev/docs/validation . The local public engine is pinned; the actual deployed server version is not asserted.
