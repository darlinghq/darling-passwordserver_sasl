project(darling-passwordserver_sasl)

#
# common settings
#
include_directories(
	cyrus_sasl
	cyrus_sasl/include
	cyrus_sasl/lib
)

add_compile_definitions(
	PIC
	HAVE_DLFCN_H
	AUTHDATA
)

#
# sasl
#

set(DYLIB_INSTALL_NAME "/usr/lib/libsasl2.2.dylib")
set(DYLIB_COMPAT_VERSION "3.0.0")
set(DYLIB_CURRENT_VERSION "3.15")

add_darling_library(sasl
	cyrus_sasl/lib/auxprop.c
	cyrus_sasl/lib/canonusr.c
	cyrus_sasl/lib/checkpw.c
	cyrus_sasl/lib/client.c
	cyrus_sasl/lib/common.c
	cyrus_sasl/lib/config.c
	cyrus_sasl/lib/dlopen.c
	cyrus_sasl/lib/external.c
	cyrus_sasl/lib/md5.c
	cyrus_sasl/lib/saslutil.c
	cyrus_sasl/lib/server.c
	cyrus_sasl/lib/seterror.c
	cyrus_sasl/lib/snprintf.c
	cyrus_sasl/plugins/plugin_common.c
)
make_fat(sasl)

set_target_properties(sasl PROPERTIES
	OUTPUT_NAME "sasl2.2"
)

target_link_libraries(sasl PRIVATE
	system
)

target_link_options(sasl PRIVATE
	-Wl,-exported_symbols_list,${CMAKE_CURRENT_SOURCE_DIR}/cyrus_sasl/lib/sasl-lib.exp
	-Wl,-dead_strip
)

install(TARGETS sasl DESTINATION libexec/darling/usr/lib)
foreach(VER IN ITEMS 2.2.0.22 2.2.0.21 2.2.0.15 2 2.2.0.1)
	InstallSymlink("libsasl2.2.dylib" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/libsasl${VER}.dylib")
endforeach()

#
# sasldb
#

#add_darling_static_library(sasldb FAT
#	cyrus_sasl/sasldb/allockey.c
#	cyrus_sasl/sasldb/db_ndbm.c
#)

#
# plugins
#

include(darling_bundle)

function(add_sasl_plugin name init_name prefix output_name suffix)
	set(DYLIB_INSTALL_NAME "/usr/lib/sasl2/${prefix}${output_name}${suffix}")
	set(DYLIB_COMPAT_VERSION "")
	set(DYLIB_CURRENT_VERSION "")
	if (NOT init_name)
		set(init_name "${name}_init")
	endif ()
	add_darling_bundle(sasl_${name} ""
		cyrus_sasl/plugins/plugin_common.c
		cyrus_sasl/plugins/${init_name}.c
		cyrus_sasl/plugins/${name}.c
		${ARGN}
	)
	set_target_properties(sasl_${name} PROPERTIES
		PREFIX "${prefix}"
		OUTPUT_NAME "${output_name}"
		SUFFIX "${suffix}"
	)
	install(TARGETS sasl_${name} DESTINATION libexec/darling/usr/lib/sasl2)
	target_link_libraries(sasl_${name} PRIVATE
		system
	)
endfunction()

add_sasl_plugin(anonymous "" "lib" "anonymous.2" ".so")

add_sasl_plugin(cram "crammd5_init" "lib" "crammd5.2" ".so")

add_sasl_plugin(digestmd5 "" "lib" "digestmd5.2" ".so")

add_sasl_plugin(gssapi "gssapiv2_init" "lib" "gssapiv2.2.0.18" ".so")

add_sasl_plugin(login "" "" "login" ".so")

add_sasl_plugin(ntlm "" "lib" "ntlm" ".so")

add_sasl_plugin(plain "" "lib" "plain.2" ".so")

# disabled in Xcode; also not present on a real macOS install
#add_sasl_plugin(sasldb "" "lib" "sasldb.2" ".so")

add_sasl_plugin(gs2 "" "lib" "gs2" ".so"
	cyrus_sasl/plugins/gs2_token.c
)

add_sasl_plugin(srp "" "" "srp" ".so")

#target_link_libraries(ssl_sasldb PRIVATE
#	sasldb
#)

#
# plugin build settings
#

target_compile_definitions(sasl_cram PRIVATE
	APPLE_ALLOW_VERIFY_ONLY=1
)

target_compile_definitions(sasl_gssapi PRIVATE
	AUTHDATA=1
	K5_MAX_SSF=112
)

target_link_libraries(sasl_gssapi PRIVATE
	Kerberos
)

#
# plugin symlinks
#

InstallSymlink("libgssapiv2.2.0.18.so" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/sasl2/libgssapiv2.2.so")
InstallSymlink("../libgssapiv2.2.so" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/sasl2/libgssapiv2.2.so")

InstallSymlink("../libcrammd5.2.so" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/sasl2/libcrammd5.2.so")

InstallSymlink("../srp.so" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/sasl2/srp.so")

