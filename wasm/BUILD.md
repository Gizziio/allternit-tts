# Browser WebAssembly build of sherpa-onnx (for `Gizziio/allternit-tts`, `wasm/BUILD.md`)

This is the corresponding-source recipe for the `sherpa-onnx-wasm.{js,wasm}` that Allternit serves under
`/voice-engine/`. The wasm links espeak-ng (GPL-3.0-or-later) for Kokoro's phonemizer, so it is distributed
under the GPL-3.0-or-later; this file and the pinned upstream sources below are the source offer.

Upstream: https://github.com/k2-fsa/sherpa-onnx, tag `v1.13.8`. No source patch. Models are NOT embedded
(the app writes them into the wasm's in-memory FS at `/tmp/m/...` from its own hash-pinned downloads).
The only local edit is `make -j3` -> `make -j2` in the build script (build parallelism, not code).

Pinned inputs the CMake build downloads (hash-checked by the repo's cmake files):
- onnxruntime wasm static lib: csukuangfj/onnxruntime-libs v1.28.2 `onnxruntime-wasm-static_lib-simd-1.28.2.zip`
- espeak-ng: https://github.com/csukuangfj/espeak-ng, commit `ed530aa113046142eb5115cf2fc9157854d0ffe1`
  (SHA256 e4e262cbe34f7fe21f91f1ba3397f2728e1f30eafbae7853f2b753a9ed13f0dd)
- other deps (kaldi-native-fbank, openfst, eigen 5.0.1, json, ...) as pinned in sherpa-onnx `cmake/*.cmake`.

## Commands
```bash
git clone https://github.com/emscripten-core/emsdk.git && cd emsdk
./emsdk install 4.0.23 && ./emsdk activate 4.0.23 && source ./emsdk_env.sh
cd .. && git clone --depth 1 --branch v1.13.8 https://github.com/k2-fsa/sherpa-onnx.git && cd sherpa-onnx
sed -i 's/make -j3/make -j2/' build-wasm-simd-web.sh   # optional
./build-wasm-simd-web.sh
# output: build-wasm-simd-web/install/bin/wasm/web/sherpa-onnx-wasm-web.{js,wasm}
```
`build-wasm-simd-web.sh` configures `-DSHERPA_ONNX_ENABLE_WASM=ON -DSHERPA_ONNX_ENABLE_WASM_WEB=ON`
(target `sherpa-onnx-wasm-web`, source `wasm/nodejs/sherpa-onnx-wasm-nodejs.cc`, `wasm/web/CMakeLists.txt`:
MEMFS, `EXPORT_NAME="SherpaOnnx"`, MODULARIZE, INITIAL_MEMORY=512MB with growth, single-threaded).

## Result (macOS arm64 host, ~3 min at nice 19, -j2)
- `sherpa-onnx-wasm-web.js`  91 KB, sha256 8d0fc17cfa75b3047b8e74fcc089abfe9033dbf6d2cca7746b726d6f91360d23
- `sherpa-onnx-wasm-web.wasm` 14 MB, sha256 f0b68c134670e4b99db5fe911551f2448dad1c7a7c3f9b8552035030713e8fd7

## Files Allternit ships next to it (from the same tag, Apache-2.0)
`wasm/asr/sherpa-onnx-asr.js`, `wasm/tts/sherpa-onnx-tts.js`, `wasm/vad/sherpa-onnx-vad.js` (copied unmodified;
the vite plugin re-exports them as ES modules). The wasm is renamed `sherpa-onnx-wasm.{js,wasm}` and its glue
is told the new name via `locateFile`.
