# How to build V8

## Steps to build V8 on Windows

1. **Set proxy server**

   ```
   set HTTP_PROXY=http://your_proxy_server:8080
   set HTTPS_PROXY=http://your_proxy_server:8080
   ```

2. **Download depot_tools**

   Download the *depot_tools* [bundle](https://storage.googleapis.com/chrome-infra/depot_tools.zip) and extract it somewhere (e.g. C:\workspace) and add a `PATH` user variable like below:

   ```
   set PATH=C:\workspace\depot_tools;%PATH%
   ```

3. **Update depot_tools**

   From a `cmd.exe` shell, run the command `gclient` (without arguments). On first run, `gclient` will install all the Windows-specific bits needed to work with the code, including `msysgit` and `python`.

   ```
   gclient
   ```

4. **Install the visual studio enterprise 2017 and relevant SDK.**

5. **Set the windows toolchain environment**

   ```
   set DEPOT_TOOLS_WIN_TOOLCHAIN=0
   ```

6. **Get the V8 source code**

   Go into the directory where you want to download the V8 source into and execute the following in your terminal/shell:

   ```
   fetch v8
   cd v8
   ```

7. **Checkout a specified version**

   Navigate to the V8 source folder, checkout the specified version (e.g. 7.0.276) and update the dependencies.

   ```
   git checkout remotes/origin/7.0.276
   gclient sync
   ```

8. **Generate the build files**

   ```
   python tools/dev/v8gen.py x64.release
   python tools/dev/v8gen.py x64.deubg
   python tools/dev/v8gen.py ia32.release
   python tools/dev/v8gen.py ia32.release
   ```

   Take the building mode `x64.release` for example.

   On success, the build files, argument file `args.gn` and relevant materials are generated under

   `out.gn\x64.release`

9. **Compile V8 as dynamic library**

   Modify the default build configuration `args.gn`  file as below:

   ```
   is_debug = false
   target_cpu = "x64"
   is_component_build = true
   v8_static_library = false
   ```

   Compile the source by executing the following:

   ```
   ninja -C out.gn/x64.release
   ```

   On success, a list of dynamic library files are created, including:

   - `icui18n.dll` 

   - `icuuc.dll` 

   - `v8_libbase.dll`

   -  `v8_libplatform.dll`  

   - `v8.dll`

10. **TODO...**

## Steps to build V8 on Linux

1. **Set proxy server**

   ```
   export HTTP_PROXY=http://your_proxy_server:8080
   export HTTPS_PROXY=http://your_proxy_server:8080
   ```

2. **Download depot_tools**

   Clone the *depot_tools* repository and update the `PATH` user

   ```
   git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
   export PATH=/home/builder/disk2/v8/v8/depot_tools;$PATH
   ```

3. **Update depot_tools**

   From a `cmd.exe` shell, run the command `gclient` (without arguments). On first run, `gclient` will install all the Linux-specific bits needed to work with the code

4. **Install GCC**

   A proper version of GCC, e.g. GCC7

5. **Get the V8 source code**

   Go into the directory where you want to download the V8 source into and execute the following in your terminal/shell:

   ```
   fetch v8
   cd v8
   ```

6. **Checkout a specified version**

   Navigate to the V8 source folder, checkout the specified version (e.g. 7.0.276) and update the dependencies.

   ```
   git checkout remotes/origin/7.0.276
   gclient sync
   ```

7. **Compile V8**

   ```
   tools/dev/gm.py x64.release
   ```

   The above command actually is a wrapper for the following commands.

   ```
   1. mkdir -p out/x64.release
   
   2. echo > out/x64.release/args.gn << EOF
   is_component_build = false
   is_debug = false
   target_cpu = "x64"
   use_goma = false
   goma_dir = "None"
   v8_enable_backtrace = true
   v8_enable_disassembler = true
   v8_enable_object_print = true
   v8_enable_verify_heap = true
   EOF
   
   3. gn gen out/x64.release
   
   4. autoninja -C out/x64.release d8
   ```

   which is to create output folder, generate the default build argument file, generate build files and start the build process: ninja -C out.gn/x64.release

8. **Compile V8 as dynamic library**

   With the the default build argument file, only static libraries`libv8_libbase.a`, `libv8_libplatform.a` and `d8` are generated. To build V8 as a dynamic library, apply below changes to file `out/x64.release/args.gn`

   ```
   is_component_build = true
   v8_static_library = false
   ```

   then run the build again:

   ```
   tools/dev/gm.py x64.release
   tools/dev/gm.py x64.debug
   ```

   On success, a list of so files are created, including:

   - `libc++.so`,

   - `libicui18n.so` 

   - `libicuuc.so` 

   - `libv8_libbase.so`

   -  `libv8_libplatform.so`  

   - `libv8.so`

     

9. **Make V8 depending on `libstdc++.so` instead of `libc++.so`**

   By default, V8 depends on `libc++.so`. 

   ```
   builder@linux-sl:~/disk2/v8/v8/out/x64.release> LD_LIBRARY_PATH=. ldd libv8.so
   	linux-vdso.so.1 (0x00007fffab1c3000)
   	libicui18n.so => ./libicui18n.so (0x00007f5182f88000)
   	libicuuc.so => ./libicuuc.so (0x00007f51841de000)
   	libv8_libbase.so => ./libv8_libbase.so (0x00007f51841be000)
   	libc++.so => ./libc++.so (0x00007f5182eb2000)
   	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5182c94000)
   	librt.so.1 => /lib64/librt.so.1 (0x00007f5182a8c000)
   	libm.so.6 => /lib64/libm.so.6 (0x00007f5182741000)
   	libc.so.6 => /lib64/libc.so.6 (0x00007f5182387000)
   	/lib64/ld-linux-x86-64.so.2 (0x00007f5184163000)
   	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f518216f000)
   ```

   However, for most GNU libraries, they depends on `libstdc++. so. `Mixing `libc++.so` and `libstdc++.so` in one executable might cause conflict issues. It is better if V8 could depend on `libstdc++.so` as well. To make it, another three flags need to be added to the args.gn file

   ```
   use_custom_libcxx=false
   use_glib=false
   use_sysroot=false
   ```

   so the complete recommend `args.gn` file is 

   ```
   is_component_build = true
   v8_static_library = false
   is_debug = false
   target_cpu = "x64"
   use_goma = false
   goma_dir = "None"
   v8_enable_backtrace = true
   v8_enable_disassembler = true
   v8_enable_object_print = true
   v8_enable_verify_heap = true
   use_custom_libcxx=false
   use_glib=false
   use_sysroot=false
   ```

   Then run the build again:

   ```
   tools/dev/gm.py x64.release
   ```

   On success, `libc++.so` is not generated anymore. Check the dependency and it can be seen `libv8.so` depends on `libstdc++.so`.

   ```
   builder@linux-sl:~/disk2/v8/v8/out/x64.release> LD_LIBRARY_PATH=. ldd libv8.so
   	linux-vdso.so.1 (0x00007fffc7399000)
   	libicui18n.so => ./libicui18n.so (0x00007f529f6ce000)
   	libicuuc.so => ./libicuuc.so (0x00007f52a08f9000)
   	libv8_libbase.so => ./libv8_libbase.so (0x00007f52a08dc000)
   	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f529f4b0000)
   	librt.so.1 => /lib64/librt.so.1 (0x00007f529f2a8000)
   	libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x00007f529ef1e000)
   	libm.so.6 => /lib64/libm.so.6 (0x00007f529ebd3000)
   	libc.so.6 => /lib64/libc.so.6 (0x00007f529e819000)
   	/lib64/ld-linux-x86-64.so.2 (0x00007f52a087e000)
   	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f529e601000)
   ```

10. **Get the build result**

    - `libicui18n.so`

    -  `libicuuc.so`
    
    -  `libv8_libbase.so` 
    
    -  `libv8_libplatform.so`
    
    -  `libv8.so`
    
    We will use these files when embedding v8 into our application.

## Reference

More details, please see:

- https://v8.dev/docs/build-gn

- https://v8.dev/docs/build

- https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up
- https://medium.com/angular-in-depth/how-to-build-v8-on-windows-and-not-go-mad-6347c69aacd4
