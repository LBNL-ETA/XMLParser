set( CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE ON )

add_library( ${LIB_NAME} 
		xmlParser.h
		xmlParser.cpp)

if(NOT "${fPIC}")
  set(fPIC ON)
endif()

if(fPIC)
  set_property(TARGET ${LIB_NAME} PROPERTY POSITION_INDEPENDENT_CODE ON)
endif()



if (CMAKE_COMPILER_IS_GNUCC)
    target_compile_options(${LIB_NAME} PRIVATE -Wall -Wextra -pedantic)
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(${LIB_NAME} PRIVATE -Wall -Wextra -pedantic )
endif()
if (MSVC)
    target_compile_options(${LIB_NAME} PRIVATE /W4)
endif()


target_compile_features(${LIB_NAME} PRIVATE cxx_std_17)
