# opentee with cmake

## build step

```sh
git clone https://github.com/code4lala/opentee cmake
ln -s cmake/CMakeLists.txt .

cmake -B build -G Ninja .

# build mbed first
cmake --build build --target mbed

# build all
cmake --build build
```

## run step

```sh
export OPENTEE_BUILD_PATH=$(readlink -f build)

# paths in opentee.conf must be absolute path
cat > $OPENTEE_BUILD_PATH/opentee.conf << EOF
[PATHS]
ta_dir_path = $OPENTEE_BUILD_PATH
core_lib_path = $OPENTEE_BUILD_PATH
subprocess_manager = libManagerApi.so
subprocess_launcher = libLauncherApi.so
EOF

# tee engine will load mbed
export LD_LIBRARY_PATH=$OPENTEE_BUILD_PATH/mbed/lib

# configuration file path must be absolute path
build/opentee-engine -c $OPENTEE_BUILD_PATH/opentee.conf
# run foreground, not as a daemon
build/opentee-engine -c $OPENTEE_BUILD_PATH/opentee.conf -f

# run test
build/conn_test
build/example_sha1
```

## misc

```sh
# check background tee daemon cmd:
ps aux | grep -i tee
# kill background tee daemon cmd:
kill $(cat /tmp/opentee/opentee-engine.pid)
```

```sh
# check ta log
tail -f /var/log/syslog
```

## coverage

https://github.com/bilke/cmake-modules/blob/master/CodeCoverage.cmake
https://github.com/StableCoder/cmake-scripts/blob/main/code-coverage.cmake
https://chromium.googlesource.com/chromium/llvm-project/libcxx/+/refs/heads/master/cmake/Modules/CodeCoverage.cmake

https://github.com/StableCoder/cmake-scripts/blob/main/code-coverage.cmake

https://clang.llvm.org/docs/SourceBasedCodeCoverage.html

llvm-profdata merge -sparse *.profraw -o default.profdata
llvm-cov show ./binary -instr-profile=default.profdata

## log pid tid

libtee/include/tee_logging.h

```c
/*!
  Print message to syslog with additional information.
*/
#define OT_LOG(level, message, ...)                                                                \
	syslog(level, DBG_LOCATION "[pid:%d][tid:%d]  " message, __FILE__, __func__, __LINE__, getpid(), gettid(), ##__VA_ARGS__)
```
