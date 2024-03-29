# Fuzzing WASM based on semantic preservation.

## Instrument each JS engine in named browsers.

  - [Manual setting in Ubuntu-22.04-x86/64](https://github.com/UsQuake/JSinRust/blob/main/embed_inst_js.md)
  - [Using dockerfiles](https://github.com/UsQuake/FuzzWasmInRust/blob/main/using_container.md)

      
## Reference for design fuzzing algorithm

  - [Analysis of chromium wasm vulnerability](https://kiss.kstudy.com/Detail/Ar?key=3921110)
  - [DIE framework for fuzzing JS](https://taesoo.kim/pubs/2020/park:die.pdf)
  - [Fuzzing wasi & browsers](https://i.blackhat.com/USA-22/Wednesday/US-22-Ventuzelo-A-Journey-Into-Fuzzing-WebAssembly-Virtual-Machines.pdf)
  - [Fuzzing wasi](https://i.blackhat.com/USA-22/Wednesday/US-22-Hai-Is-WebAssembly-Really-Safe-wp.pdf)

## Fuzzing harness design

  - ![WASM_fuzzing_engine](https://github.com/UsQuake/FuzzWasmInRust/assets/24998577/61d02b69-b0d2-42f2-9979-86fe9eaac694)
