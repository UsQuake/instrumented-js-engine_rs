
  # Embed AFL++(CLASSIC mode) instrumented javascript engine in rust code.

  ## 0. Supported environment
  
  - Now, x86 Ubuntu-22.02 only.
    
  - We will support more environment after research experiment is done.
  
  - Even though WSL is unsupported.
    * Because of linux file-system is different per environment.
  
  ## 1. Set AFL++ & LLVM.

  - For support AFL++ LLVM mode, get both llvm-15 and llvm-15-tools.
    + `sudo apt-get install -y llvm-15 llvm-15-tools`
    + Why use 15? 
      * Because llvm-15 is current version of llvm in Ubuntu-22.0.2 and you can use amazing features of AFL++ with current llvm.
    + If new version of llvm is coming out, You can consider new version. 
    
  - Clone *AFL++*.
     + `cd path_to_clone`
     + `git clone https://github.com/AFLplusplus/AFLplusplus.git`

  - Compile AFL++ with llvm-*INSTALL-VERSION*.
     + For an example, I installed llvm-15 & llvm-15-tools and I'll use them.
     + `cd path_to_clone/AFLplusplus`
     + `export LLVM_CONFIG=llvm-config-15`
     + `make distrib`
      
  - Install compiled AFL++ with root permission.
     + `sudo make install`

  - Check whether afl-clang-fast is installed.
     + `afl-clang-fast -hh`
     + `which afl-clang-fast`
   
  - Make an static library archive of afl-llvm-rt.o
     + `ar libafl-rt.a afl-llvm-rt.o`
       
  - Make an symbolic-link of afl-clang-fast to hook clang/clang++ command.
     + `cd usr/local/bin`
     + `ln -s afl-clang-fast clang`
     + `ln -s afl-clang-fast++ clang++`
       
 - Make an hard-link(copy) of llvm-ar-15 to hook llvm-ar command.
     + `cd /usr/bin`
     + `cp llvm-ar-15 /usr/local/bin/llvm-ar`
       
  ## 2. Install and set build environment to build JS engine.
  
  ### V8
  
  - Clone *rusty_v8* which is an Rust-FFI of v8 static library.
     + `cd path_to_clone`
     + `git clone https://github.com/denoland/rusty_v8.git`
        
  - Install build dependencies.
     + `sudo apt install -y libglib2.0-dev`

  ### SpiderMonkey
  
  - Clone *mozjs* which is an Rust-FFI of SpiderMonkey static library.
     + `cd path_to_clone`
     + `git clone https://github.com/servo/mozjs.git`

  - Edit build script not to instrument glue code for rust to c++.
     + `cd path_to_clone/mozjs/rust-mozjs`
     + Under 15th line in *build.rs*, add follow line.
     + `.compile(clang++-15)`
    
  ### JavascriptCore
  
  - Clone *jsc-sys* which is an Rust-FFI of jsc static library.
     + `cd path_to_clone`
     + `git clone https://github.com/drtychai/jsc-sys.git`

  - Clone WebKit *inside* of jsc-sys repo.
     + `cd path_to_clone/jsc-sys`
     + `git clone https://github.com/WebKit/WebKit.git`
   
  - Edit build script.
    
     + Under 55th line in *makefile.cargo*, add follow option. (*path_to_python* is path to python binary.)
     + `-DPython_EXECUTABLE=path_to_python`

     + Under 103th line in *build.rs*, add follow option.
     + `println!("cargo:rustc-link-lib=atomic");`
       
  ## 3. Set build option.
  
  ### Common options.
  
  - *path_to_clone_AFL++* is path to clone AFLplusplus.
  - `export AFL_PATH="path_to_clone_AFL++" AFL_LLVM_INSTRUMENT="CLASSIC" RUSTFLAGS="-Ccodegen-units=1 -Clink-arg=-fuse-ld=gold -lafl-rt -Lpath_to_clone_AFL++"`

  ### V8
  
  - *path_to_python3* is path to Python3 executable.
  - `export CLANG_BASE_PATH="/usr/local" V8_FROM_SOURCE=1 PYTHON="path_to_python3"`

  ### MozJS
  
  - /usr/lib/x86_64-linux-gnu is path of libclang-version.so in Ubuntu-X86.
  - `export CC="afl-clang-fast" CXX="afl-clang-fast++" LIBCLANG_PATH=/usr/lib/x86_64-linux-gnu`  
     
  ### JSC
  
  - Nothing to do. because, we already edit the makefile.

 ## 4. Clone and build own runners

 ### Write your own runner

 - If you want to write own runner,
 - Edit *Cargo.toml* of each runner to use your custom build of rusty JS engines.
     
 ### V8 sample runner
  
  - Clone *v8-integ* which is an sample runner.
     + `cd path_to_clone`
     + `git clone https://github.com/UsQuake/v8_integ.git`

  - Edit *Cargo.toml* to use custom build patch.
     + `cd path_to_clone/v8_integ`
     + Add follow line in *Cargo.toml* under '[dependencies]'.
     + ```v8 = {path ="path_to_clone_rusty_v8"}```
     + *path_to_clone_rusty_v8* is path to cloned & customed rusty v8.

  - Finally, build runner.
     + `cargo build`
       
 ### SpiderMonkey sample runner
  
  - Clone *mozjs-integ* which is an sample runner.
     + `cd path_to_clone`
     + `git clone https://github.com/UsQuake/mozjs_integ.git`

  - Edit *Cargo.toml* to use custom build patch.
     + `cd path_to_clone/mozjs_integ`
     + Add follow lines in *Cargo.toml* under '[dependencies]'.
     + ```
       mozjs = {path ="path_to_clone_mozjs/rust-mozjs"}
       [patch."https://github.com/servo/mozjs"]
       mozjs_sys = { path = "path_to_clone_mozjs/mozjs" }
       ```
     + *path_to_clone_mozjs* is path to cloned & customed mozjs.
       
  - Finally, build runner.
     + `cargo build`
         
 ### JSC runner
  
  - Clone *jsc-integ* which is an sample runner.
     + `cd path_to_clone`
     + `git clone https://github.com/UsQuake/jsc_integ.git`
       
  - Edit *Cargo.toml* to use custom build patch.
     + `cd path_to_clone/jsc_integ`
     + Add follow lines in *Cargo.toml* under '[dependencies]'.
     + ```
       jscjs_sys = {path ="path_to_clone_jsc_sys"}
       url = "2.4.0"
       ```
     + *path_to_clone_jsc_sys* is path to cloned & customed jsc_sys.

   - Finally, build runner.
     + `cargo build`