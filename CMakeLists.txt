cmake_minimum_required(VERSION 3.18)
project(opentee)


add_library(CommonApi SHARED
    emulator/common/com_protocol.c
    emulator/common/elf_read.c
    emulator/common/epoll_wrapper.c
    emulator/common/tee_list.c
    emulator/common/tee_logging.c
)
target_include_directories(CommonApi PUBLIC
    emulator/include
    emulator/internal_api
)
target_link_libraries(CommonApi PUBLIC
    elf
    z
)


add_executable(opentee-engine
    emulator/opentee-main/args.c
    emulator/opentee-main/conf_parser.c
    emulator/opentee-main/ini.c
    emulator/opentee-main/main.c
)
target_include_directories(opentee-engine PUBLIC
    emulator/include
    emulator/opentee-main
)
target_link_options(opentee-engine PUBLIC
    -rdynamic
)
target_link_libraries(opentee-engine PUBLIC
    pthread
    dl

    CommonApi
)

set(MBED_INSTALL_DIR ${CMAKE_BINARY_DIR}/mbed)
set(MBED_INCLUDE_DIR ${MBED_INSTALL_DIR}/include)
set(MBED_LIBRARY_DIR ${MBED_INSTALL_DIR}/lib)
include(ExternalProject)
ExternalProject_Add(mbed
    SOURCE_DIR        "${CMAKE_SOURCE_DIR}/../mbedtls-3.1.0"
    CMAKE_ARGS        -DCMAKE_INSTALL_PREFIX:PATH=${MBED_INSTALL_DIR} -DUSE_SHARED_MBEDTLS_LIBRARY=On
)


add_library(InternalApi SHARED
    emulator/internal_api/callbacks.c
    emulator/internal_api/opentee_internal_api.c
    emulator/internal_api/tee_bigint.c
    emulator/internal_api/tee_cancellation.c
    emulator/internal_api/tee_internal_client_api.c
    emulator/internal_api/tee_memory.c
    emulator/internal_api/tee_panic.c
    emulator/internal_api/tee_time_api.c
    emulator/internal_api/crypto/crypto_ae.c
    emulator/internal_api/crypto/crypto_asym.c
    emulator/internal_api/crypto/crypto_cipher.c
    emulator/internal_api/crypto/crypto_digest.c
    emulator/internal_api/crypto/crypto_mac.c
    emulator/internal_api/crypto/crypto_utils.c
    emulator/internal_api/crypto/crypto_key_derive.c
    emulator/internal_api/crypto/crypto_generate_random.c
    emulator/internal_api/storage/data_stream_api.c
    emulator/internal_api/storage/enumerator_api.c
    emulator/internal_api/storage/persistent_object_api.c
    emulator/internal_api/storage/storage_utils.c
    emulator/internal_api/storage/transient_object_api.c
    emulator/internal_api/crypto/tee_crypto_api.c
    emulator/internal_api/storage/tee_storage_api.c
)
add_dependencies(InternalApi mbed)
target_include_directories(InternalApi PUBLIC
	emulator/include
	emulator/internal_api
    ${MBED_INCLUDE_DIR}
)
target_link_libraries(InternalApi PUBLIC
    mbedcrypto

    CommonApi
)
target_link_options(InternalApi PUBLIC
    -L${MBED_LIBRARY_DIR}
)


add_library(LauncherApi SHARED
    emulator/launcher/dynamic_loader.c
    emulator/launcher/launcher_mainloop.c
    emulator/launcher/ta_internal_thread.c
    emulator/launcher/ta_io_thread.c
    emulator/launcher/ta_process.c
    emulator/launcher/ta_signal_handler.c
)
target_include_directories(LauncherApi PUBLIC
	emulator/include
	emulator/internal_api
	emulator/launcher
)
target_link_libraries(LauncherApi PUBLIC
    pthread
    dl
    rt

    InternalApi
    CommonApi
)


add_library(ManagerApi SHARED
    emulator/manager/ext_storage_stream_api_posix.c
    emulator/manager/opentee_manager_storage_api.c
    emulator/manager/io_thread.c
    emulator/manager/logic_thread.c
    emulator/manager/mainloop.c
    emulator/manager/ta_dir_watch.c
    emulator/manager/shm_mem.c
)
target_include_directories(ManagerApi PUBLIC
	emulator/include
    emulator/manager
	emulator/internal_api
    ${MBED_INCLUDE_DIR}
)
target_link_libraries(ManagerApi PUBLIC
    dl
    pthread
    rt

    CommonApi
)

add_library(tee SHARED
    libtee/src/com_protocol.c
    libtee/src/tee_client_api.c
)
target_include_directories(tee PUBLIC
    libtee/include
    libtee/src
)
target_link_libraries(tee PUBLIC
    z
)


add_library(omnishare SHARED
    libomnishare/src/omnishare.c
)
target_include_directories(omnishare PUBLIC
	libomnishare/include
	libomnishare/src
	libtee/include
)
target_link_libraries(omnishare PUBLIC
    tee
)


add_library(tee_pkcs11 SHARED
    libtee_pkcs11/src/hal_gp.c
    libtee_pkcs11/src/mutex_manager.c
    libtee_pkcs11/src/pkcs11_crypto.c
    libtee_pkcs11/src/pkcs11_general.c
    libtee_pkcs11/src/pkcs11_object.c
    libtee_pkcs11/src/pkcs11_session_slot.c 
)
target_include_directories(tee_pkcs11 PUBLIC
	libtee_pkcs11/include
	libtee_pkcs11/src
	libtee/include
)
target_link_libraries(tee_pkcs11 PUBLIC
    tee
)

add_library(CryptoTest SHARED
	tests/internal_api/crypto_test.c
)
target_include_directories(CryptoTest PUBLIC
    emulator/include
)
target_link_libraries(CryptoTest PUBLIC
    InternalApi
)


add_executable(services_test
    CAs/services_test/services_test.c
)
target_include_directories(services_test PUBLIC
    libtee/include
)
target_link_options(services_test PUBLIC
    -rdynamic
)
target_link_libraries(services_test PUBLIC
    tee
)


add_executable(example_sha1
    CAs/example_sha1_ca/example_sha1_ca.c
)
target_include_directories(example_sha1 PUBLIC
    libtee/include
)
target_link_options(example_sha1 PUBLIC
    -rdynamic
)
target_link_libraries(example_sha1 PUBLIC
    tee
)

add_executable(usr_study
    CAs/usr_study_ca/usr_study_ca.c
)
target_include_directories(usr_study PUBLIC
    libtee/include
)
target_link_options(usr_study PUBLIC
    -rdynamic
)
target_link_libraries(usr_study PUBLIC
    tee
)


add_executable(sign_non_tee_ecdsa_256
    CAs/example_conversion/sign_non_tee_ecdsa_256.c
)
target_include_directories(sign_non_tee_ecdsa_256 PUBLIC
    libtee/include
    ${MBED_INCLUDE_DIR}
)
target_link_options(sign_non_tee_ecdsa_256 PUBLIC
    -rdynamic
)
target_link_options(sign_non_tee_ecdsa_256 PUBLIC
    -L${MBED_LIBRARY_DIR}
)
target_link_libraries(sign_non_tee_ecdsa_256 PUBLIC
    mbedcrypto
)


add_executable(sign_tee_ecdsa_256
    CAs/example_conversion/sign_tee_ecdsa_256.c
)
target_include_directories(sign_tee_ecdsa_256 PUBLIC
    libtee/include
)
target_link_options(sign_tee_ecdsa_256 PUBLIC
    -rdynamic
)
target_link_libraries(sign_tee_ecdsa_256 PUBLIC
    tee
)

add_executable(conn_test
    CAs/conn_test_app/conn_test_app.c
)
target_include_directories(conn_test PUBLIC
    libtee/include
)
target_link_options(conn_test PUBLIC
    -rdynamic
)
target_link_libraries(conn_test PUBLIC
    tee
)

add_executable(omnishare-fuse
    CAs/omnishare_fuse/omnishare_fuse.c
)
target_compile_definitions(omnishare-fuse PUBLIC
    _FILE_OFFSET_BITS=64
)
target_compile_options(omnishare-fuse PUBLIC
    -DOT_LOGGING
    -DFUSE_USE_VERSION=26
)
target_include_directories(omnishare-fuse PUBLIC
    CAs/libomnishare/include
)
target_link_options(omnishare-fuse PUBLIC
    -rdynamic
)
target_link_libraries(omnishare-fuse PUBLIC
    fuse

    omnishare
)


add_library(omnishare_ta SHARED
    TAs/omnishare_ta/omnishare_ta.c
)
target_include_directories(omnishare_ta PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(omnishare_ta PUBLIC
    TA_PLUGIN
)


add_library(pkcs11_ta SHARED
    TAs/pkcs11_ta/common/compat.c
    TAs/pkcs11_ta/gp/pkcs11_ta.c
    TAs/pkcs11_ta/gp/crypto.c
    TAs/pkcs11_ta/gp/object.c
    TAs/pkcs11_ta/gp/open_tee_conf.c
    TAs/pkcs11_ta/gp/pkcs11_application.c
    TAs/pkcs11_ta/gp/pkcs11_session.c
    TAs/pkcs11_ta/gp/slot_token.c
    TAs/pkcs11_ta/gp/utils.c
)
target_include_directories(pkcs11_ta PUBLIC
    emulator/include
    TAs/pkcs11_ta/common
    TAs/pkcs11_ta/gp
    TAs/include
)
target_compile_definitions(pkcs11_ta PUBLIC
    TA_PLUGIN
)


add_library(ta_services SHARED
    TAs/ta_services/ta_services.c
)
target_include_directories(ta_services PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(ta_services PUBLIC
    TA_PLUGIN
)

add_library(example_digest SHARED
    TAs/example_digest_ta/example_digest_ta.c
)
target_include_directories(example_digest PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(example_digest PUBLIC
    TA_PLUGIN
)

add_library(sign_ecdsa_256 SHARED
    TAs/example_conversion/sign_ecdsa_256.c
)
target_include_directories(sign_ecdsa_256 PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(sign_ecdsa_256 PUBLIC
    TA_PLUGIN
)

add_library(user_study SHARED
    TAs/usr_study_ta/usr_study_ta.c
)
target_include_directories(user_study PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(user_study PUBLIC
    TA_PLUGIN
)

add_library(ta2ta_conn_test SHARED
    TAs/ta2ta_conn_test_app/ta2ta_conn_test_app.c
)
target_include_directories(ta2ta_conn_test PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(ta2ta_conn_test PUBLIC
    TA_PLUGIN
)



add_library(StorageTest SHARED
    tests/internal_api/storage_test.c
)
target_include_directories(StorageTest PUBLIC
    emulator/include
)
target_link_libraries(StorageTest PUBLIC
    InternalApi
)


add_library(ta2taTest SHARED
    tests/internal_api/ta2ta_test.c
)
target_include_directories(ta2taTest PUBLIC
    emulator/include
    TAs/ta2ta_conn_test_app
)
target_link_libraries(ta2taTest PUBLIC
    InternalApi
)


add_library(ta_conn_test SHARED
    TAs/ta_conn_test_app/ta_conn_test_app.c
)
target_include_directories(ta_conn_test PUBLIC
    emulator/include
    tests/internal_api
    TAs/include
)
target_link_libraries(ta_conn_test PUBLIC
    CryptoTest
    StorageTest
    ta2taTest
)
target_compile_definitions(ta_conn_test PUBLIC
    TA_PLUGIN
)


add_library(ta_panic_crash SHARED
    TAs/ta_panic_crash/ta_panic_crash.c
)
target_include_directories(ta_panic_crash PUBLIC
    emulator/include
    tests/internal_api
    TAs/include
)
target_link_libraries(ta_panic_crash PUBLIC
    CryptoTest
    StorageTest
    ta2taTest
)
target_compile_definitions(ta_panic_crash PUBLIC
    TA_PLUGIN
)


add_library(example_ta SHARED
    TAs/example_ta/example_ta.c
    TAs/example_ta/open_tee_conf.c
)
target_include_directories(example_ta PUBLIC
    emulator/include
    TAs/include
)
target_compile_definitions(example_ta PUBLIC
    TA_PLUGIN
)



add_executable(pkcs11_test
    tests/pkcs11/pkcs11_test_app.c
)
target_include_directories(pkcs11_test PUBLIC
    emulator/include
    libtee_pkcs11/include
)
target_link_libraries(pkcs11_test PUBLIC
    tee
    tee_pkcs11
)


add_definitions(-DOT_LOGGING)