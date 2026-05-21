# fpsample-wheels

Republished prebuilt wheels for [`fpsample`](https://github.com/leonardodalinky/fpsample) v1.0.2 covering Python versions upstream does not yet ship to PyPI.

## Why

Upstream `fpsample` 1.0.2 publishes binary wheels for cp38–cp312 on `linux_x86_64` and `win_amd64`. No cp313 or cp314 yet. ComfyUI-Hunyuan3D-Part's pixi env (managed by `comfy-env`) randomly picks Python 3.10–3.14, and on the cp313/cp314 picks `uv` falls back to source-building fpsample, which (depending on platform) drags in a Rust+CMake toolchain that's not always available on our CI runners.

This repo builds fpsample v1.0.2 from upstream source via `cibuildwheel` for every Python version we test against and publishes the wheels to a GitHub Release per tag. Downstream pixi envs add this index as an extra wheel source so the resolver never has to source-build fpsample.

## What's in the box

| Platform | Python versions |
|---|---|
| `linux_x86_64` (manylinux 2_28) | cp310, cp311, cp312, cp313, cp314 |
| `win_amd64` | cp310, cp311, cp312, cp313, cp314 |

No macOS — we don't run macOS in CI for any of the nodes that pull fpsample.

## How to consume

Pin upstream version (1.0.2) and add this release as a pip `--find-links` source:

```bash
pip install --find-links \
  https://github.com/PozzettiAndrea/fpsample-wheels/releases/expanded_assets/v1.0.2 \
  fpsample==1.0.2
```

Or, in a pixi env's `comfy-env.toml`, add the release URL via `[pypi-options]`:

```toml
[pypi-options]
find-links = [
  "https://github.com/PozzettiAndrea/fpsample-wheels/releases/expanded_assets/v1.0.2",
]

[pypi-dependencies]
fpsample = "==1.0.2"
```

## Build / release

The fpsample source tree is vendored as a git submodule pinned to upstream's `v1.0.2` tag. To cut a new release:

1. Update the submodule to the new upstream tag:

   ```bash
   git submodule update --remote fpsample
   cd fpsample && git checkout v<new-tag> && cd ..
   git add fpsample
   git commit -m "fpsample -> v<new-tag>"
   ```

2. Tag the parent repo (matching upstream's version, but prefixed with `v`):

   ```bash
   git tag v1.0.2 && git push origin v1.0.2
   ```

3. The `Build wheels` workflow runs on every `v*.*.*` tag, builds the 10-cell matrix (2 OS × 5 python), and publishes wheels to the release.

## Licensing

This repo only contains build glue (workflow YAML, README, .gitmodules) and a submodule pointer. The fpsample source code itself is unmodified and remains under its upstream MIT license.
