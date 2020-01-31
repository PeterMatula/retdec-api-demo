
include(ExternalProject)

find_package(Threads REQUIRED)
find_package(ZLIB REQUIRED)

if(CMAKE_C_COMPILER)
    set(CMAKE_C_COMPILER_OPTION "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
endif()
if(CMAKE_CXX_COMPILER)
    set(CMAKE_CXX_COMPILER_OPTION "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
endif()

ExternalProject_Add(retdec-project
	URL https://github.com/avast/retdec/archive/c30c33ef15dd1d2114b8fddf702e2d8e72604555.zip
	URL_HASH SHA256=d942b5ead5679e393457aa695a2cee8cc963cb51121c7721aeb35a98ad2c3e5c
	DOWNLOAD_NAME retdec.zip
	CMAKE_GENERATOR ${CMAKE_GENERATOR}
	CMAKE_GENERATOR_PLATFORM ${CMAKE_GENERATOR_PLATFORM}
	CMAKE_ARGS
		-D CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
		# Enable only those RetDec components that are actually used.
		-D RETDEC_ENABLE_RETDEC=ON
		-D RETDEC_ENABLE_RETDECTOOL=ON
		# Force the use of the same compiler as used to build the top-level
		# project. Otherwise, the external project may pick up a different
		# compiler, which may result in link errors.
		"${CMAKE_C_COMPILER_OPTION}"
		"${CMAKE_CXX_COMPILER_OPTION}"
	# Disable the install step.
	INSTALL_COMMAND ""
	LOG_DOWNLOAD ON
	LOG_CONFIGURE ON
	LOG_BUILD ON
)

ExternalProject_Get_Property(retdec-project source_dir)
ExternalProject_Get_Property(retdec-project binary_dir)

set(LLVM_LIB_LIST
	LLVMDebugInfoDWARF
	LLVMBitWriter
	LLVMIRReader
	LLVMObject
	LLVMBinaryFormat
	LLVMInstCombine
	LLVMSupport
	LLVMDemangle
	LLVMipo
	LLVMAsmParser
	LLVMBitReader
	LLVMMCParser
	LLVMCodeGen
	LLVMScalarOpts
	LLVMTransformUtils
	LLVMAnalysis
	LLVMTarget
	LLVMCore
	LLVMMC
	LLVMObject
	LLVMPasses
)

add_library(retdec-api INTERFACE)
add_dependencies(retdec-api retdec-project)
target_include_directories(retdec-api SYSTEM INTERFACE
	${source_dir}/include
	${source_dir}/include/retdec
	${binary_dir}/external/src/capstone-project/include
	${binary_dir}/external/src/llvm-project/include
	${binary_dir}/external/src/llvm-project-build/include
)

target_link_libraries(retdec-api INTERFACE optimized
	${binary_dir}/src/retdec/libretdec-retdec${CMAKE_STATIC_LIBRARY_SUFFIX}
	-Wl,--whole-archive
	${binary_dir}/src/bin2llvmir/libretdec-bin2llvmir${CMAKE_STATIC_LIBRARY_SUFFIX}
	-Wl,--no-whole-archive

	${binary_dir}/src/rtti-finder/libretdec-rtti-finder${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/loader/libretdec-loader${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/fileformat/libretdec-fileformat${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/debugformat/libretdec-debugformat${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/capstone2llvmir/libretdec-capstone2llvmir${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/stacofin/libretdec-stacofin${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/llvm-support/libretdec-llvm-support${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/config/libretdec-config${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/serdes/libretdec-serdes${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/common/libretdec-common${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/crypto/libretdec-crypto${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/ctypesparser/libretdec-ctypesparser${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/demangler/libretdec-demangler${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/ctypesparser/libretdec-ctypesparser${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/ctypes/libretdec-ctypes${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/utils/libretdec-utils${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/pdbparser/libretdec-pdbparser${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/pelib/libpelib${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/src/yaracpp/libretdec-yaracpp${CMAKE_STATIC_LIBRARY_SUFFIX}

	${binary_dir}/external/src/capstone-project-build/libcapstone${CMAKE_STATIC_LIBRARY_SUFFIX}
	${binary_dir}/deps/yara/yara/src/yara/libyara/.libs/libyara${CMAKE_STATIC_LIBRARY_SUFFIX}
	ssl
	crypto
	Threads::Threads
	${ZLIB_LIBRARIES}
	${CMAKE_DL_LIBS}
)

foreach(LLVM_LIB ${LLVM_LIB_LIST})
	target_link_libraries(retdec-api INTERFACE optimized
		${binary_dir}/external/src/llvm-project-build/lib/${CMAKE_STATIC_LIBRARY_PREFIX}${LLVM_LIB}${CMAKE_STATIC_LIBRARY_SUFFIX}
	)
endforeach(LLVM_LIB)
