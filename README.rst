Installation
------------------
* Preparation
   - download this repository
      =========================================================
      -> git clone https://github.com/igor-1982/rest_workspace.git
      -> cd rest_workspace
      -> mkdir lib include
      -> export REST_EXT_DIR="$(pwd)/lib"
      -> export REST_EXT_INC="$(pwd)/include"
      =========================================================
   - install RUST 
      =========================================================
      The REST package is built on the Rust programming language. Therefore, installing Rust is the initial step in setting up REST.

      1. Add environment paths:
         ```bash
         export CARGO_HOME=/path/to/your/custom/location/cargo
         export RUSTUP_HOME=/path/to/your/custom/location/rustup
         # For users in China, you may want to set a mirror environment to speed up
         export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
         export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
         ```
         You can run these commands one by one in the terminal now. Please ensure to add them to your default environment, like the `.bashrc` file, in any case.
         
      2. Install from official scripts:
         ```bash 
         RUSTUP_HOME=$RUSTUP_HOME CARGO_HOME=$CARGO_HOME bash -c 'curl https://sh.rustup.rs -sSf | sh'
         ```
         - Running `curl https://sh.rustup.rs` will download the official script, which can determine your system and then install Rust.
         - If you can download the scripts from other sources, you can run the scripts directly:
            `RUSTUP_HOME=$RUSTUP_HOME CARGO_HOME=$CARGO_HOME bash rustup-init.sh ` 

      3. (optional) If you use `LMOD: module` to manage your applications, you may need the following setting in the file `rust.lua`
         ```lua
         local help_message = [[
         rust
         ]]

         help(help_message,"\n")

         whatis("Name: rust")
         whatis("Version: 1.74.0")
         whatis("Keywords: rust 1.74.0")
         whatis("Description: A shell environment.")

         family("rust")

         local prefix="/path/to/your/custom/location"
         setenv("RUSTUP_HOME", prefix .. "/rustup")
         setenv("CARGO_HOME", prefix .. "/cargo")
         
         setenv("RUSTUP_DIST_SERVER", "https://mirrors.ustc.edu.cn/rust-static")
         setenv("RUSTUP_UPDATE_ROOT","https://mirrors.ustc.edu.cn/rust-static/rustup")

         prepend_path("PATH", prefix .. "/cargo" .. "/bin")
         ```

      4. Activate your Rust installation:
         - for `bash`
         ```bash
         source $RUSTUP_HOME/cargo/env
         ```
         - for `module`
         ```bash
         module load rust
         ```
      5. Test the installation:
         ```bash 
         cargo --version
         cargo 1.74.0 (ecb9851af 2023-10-18)
         ```
         Installation is complete.
      =========================================================

* Prerequisites
   - libopenssl.so/libssl.so/libcrypto.so
     =========================================================
     -> yum install openssl
        install openssl using the package manager,
        for example yum for CentOS, apt for Ubuntu
     =========================================================
   - libopenblas.so
     =========================================================
     -> git clone https://github.com/xianyi/OpenBLAS.git OpenBLAS
     -> cd OpenBLAS
     -> make -jN DYNAMIC_ARCH=1
        The variable "N" represents a integer value that indicates the number of CPUs you wish to use for the compilation process.
        DYNAMIC_ARCH=1 is needed for the use of dftd, also make it compatible with various situations.
     -> cp libopenblas.so* $REST_EXT_DIR/libopenblas.so*
     =========================================================
   - libcint.so
     =========================================================
     -> git clone https://github.com/sunqm/libcint.git libcint
     -> cd libcint
     -> mkdir build; cd build
     -> cmake -DWITH_RANGE_COULOMB=1 ..
     -> make -jN
        The variable "N" represents a integer value that indicates the number of CPUs you wish to use for the compilation process.
     -> cp libcint.so* $REST_EXT_DIR/
     =========================================================
   - libxc.so
     =========================================================
     -> based on your os, download the installation file from 
            https://www.tddft.org/programs/libxc/download/
     -> tar xf libxc-*
     -> cd libxc-*
     -> ./configure --prefix=$(pwd) --enable-shared
          `--enable-shared` is needed to produce the share library "libxc.so", otherwise only libxc.a is generated.
     -> make -jN
        The variable "N" represents a integer value that indicates the number of CPUs you wish to use for the compilation process.
     -> make install
     -> cp lib/libxc.so* $REST_EXT_DIR
     =========================================================
   - libhdf5.so
     =========================================================
     -> download the source code from https://www.hdfgroup.org/downloads/hdf5
          DO NOT DOWNLOAD the .zip file, if you are using a Linux system.
          Please consider downloading version 1.8.x since REST calls the older version of HDF5 functions. 
          The APIs of v1.14 only contain H5L_iterate, and REST calls the function H5Literate, resulting in an error "undefined reference to 'H5Literate'". 
          Although the latest HDF5 offers the 1.8 APIs, specify --with-default-api-version=v18 when building. However, please note that this flag may not be effective in practice.
          You can run `nm -D lib/libhdf5.so | grep H5Literate` after making hdf5 compilation to check the function existance.
     -> tar -zcvf hdf5-*.tar.gz
     -> cd hdf5-*
     -> ./configure --prefix=$(pwd) 
     -> CC=`your mpicc path/mpicc` ./configure --with-default-api-version=v18 --enable-parallel  --prefix=$(pwd) 
        [optional] use of parallel HDF5.
     -> make -jN
        The variable "N" represents a integer value that indicates the number of CPUs you wish to use for the compilation process.
     -> make -jN install
     -> cp lib/libhdf5.so* $REST_EXT_DIR/
     =========================================================
   - librest2fch.so
     =========================================================
     -> git clone https://gitlab.com/jeanwsr/MOKIT -b for-rest
     -> cd MOKIT/src
     -> make rest2fch -f Makefile.gnu_openblas
     -> cp MOKIT/mokit/lib/librest2fch.so $REST_EXT_DIR/
     =========================================================
   - libs-dftd3.so
     =========================================================
     -> git clone https://github.com/dftd3/simple-dftd3.git
         Also, you can compile the DFTD3 library using the way in this repository. Here we show the cmake way. 
         Aside from cmake (version 3.14 or newer), you also need a ninja build system (version 1.10 or newer). 
         Get it from https://ninja-build.org/ or install it via your package manager.
         NOTE:: if no cmake with proper version installed in your OS, try other build systems, e.g. meson or fpm.
         More details can be found on https://github.com/dftd3/simple-dftd3
     -> cd simple-dftd3
     -> cmake -B build -G Ninja -DBUILD_SHARED_LIBS=1
     -> cmake --build build
     -> cp build/libs-dftd3.so.* $REST_EXT_DIR/
     -> mkdir -f $REST_EXT_INC/dftd3
     -> find build -name *.mod | xargs -I {} cp {} $REST_EXT_INC/dftd3
         This step is rather important to finish building the system. 
         Wherever your $REST_EXT_INC is, you should always put your mods in the dftd3 directory.
     ===========================================================
   - libdftd4.so
     ===========================================================
       -> git clone https://github.com/dftd4/dftd4.git
         The process is almost the same as the build of libs-dftd3.
         Similiarly, you can compile the DFTD4 library using the cmake build system.
         NOTE:: if no cmake with proper version installed in your OS, try other build systems, e.g. meson and fpm.
         More details can be found on https://github.com/dftd4/dftd4
       -> cd dftd4
       -> cmake -B build -G Ninja -DBUILD_SHARED_LIBS=1
       -> cmake --build build
       -> mkdir -f $REST_EXT_INC/dftd4
       -> cp build/libdftd4.so.* $REST_EXT_DIR/
       -> find build -name *.mod | xargs -I {} cp {} $REST_EXT_INC/dftd4
     ===========================================================

* Build REST

   1) -> ./Config [OPTION...] [OPTARG]
        a) prepare the relevant global environment variables heading with "REST", including:
           "REST_HOME", "REST_EXT_DIR", "REST_EXT_INC", and "REST_FORTRAN_COMPILER" 
        b) download the required sub-projects: "rest", "rest_tensors", "rest_libcint", "rest_regression"
           from either github or local repository
        For more details, please use the help option:
        -> ./Config -h
   2) Source $HOME/.bash_profile
   3) -> cargo build           (debug version: compile fast and run slowly)
      or
      -> cargo build --release (release version: compile slowly and run fast)
   4) check installation
      a) check executable file generated
         -> cd $REST_HOME/target/release # for cargo build --release
         -> cd $REST_HOME/target/debug   # for cargo build
         -> ls rest
      b) check all `lib` files are linked properly
         -> ldd target/debug/rest
            ...
            libxc.so.12 => /..your...rest...path/lib/libxc.so.12 (0x00002b4cdfa32000)
            librest2fch.so => /..your...rest...path/lib/librest2fch.so (0x00002b4cdde4a000)
            libcint.so.6 => /..your...rest...path/lib/libcint.so.6 (0x00002b4ce0f50000)
            librestmatr.so => /..your...rest...path/lib/librestmatr.so (0x00002b4cdde9b000)
            libc.so.6 => /...system path.../libc.so.6 (0x00002b4ce283f000)
            libgfortran.so.5 => /...system path.../libgfortran.so.5 (0x00002b4ce2c0d000)
            libquadmath.so.0 => /...system path.../libquadmath.so.0 (0x00002b4ce3085000)
            ...
      c) run a regression test with `rest` 
         -> $REST_HOME/target/debug/rest_regression -p $REST_HOME/target/release/rest
         for more details about `rest_regression`, please type:
         -> $REST_HOME/target/debug/rest_regression -h

* To use REST as a module in the python environment

    1) cargo build [--release]
    2) cp target debug/[release]/libpyrest.so $pyfolder/pyrest.so
    3) import pyrest as a standard python module
