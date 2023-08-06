# opentee with cmake

## build step

```sh
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

# run test
build/conn_test
```

## misc

```sh
# check background tee daemon cmd:
ps aux | grep -i tee
# kill background tee daemon cmd:
ps aux | grep -i tee | awk {'print$2'} | xargs kill -9
```

```sh
# check ta log
tail -f /var/log/syslog
```
