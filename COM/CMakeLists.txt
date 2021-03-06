cmake_minimum_required (VERSION 2.8)
project(COM)
ENABLE_LANGUAGE( CXX Fortran )
IF(NOT ENABLE_MPI)
  add_definitions( -DDUMMY_MPI )
ELSE()
  FIND_PACKAGE(MPI REQUIRED)
  add_definitions( -DMPICH_IGNORE_CXX_SEEK )
  INCLUDE_DIRECTORIES(${MPI_INCLUDE_PATH})
ENDIF()
include(FortranCInterface)
FortranCInterface_HEADER(${CMAKE_BINARY_DIR}/include/FC.h MACRO_NAMESPACE "FC_")
FortranCInterface_VERIFY(CXX QUIET)
set (BUILD_STATIC FALSE CACHE BOOL "Build static COM library")
set (ENABLE_MPI TRUE CACHE BOOL "Build with MPI Support")
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single directory for all executables.")
set (LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib CACHE PATH "Single directory for all libraries and archives.")
mark_as_advanced (LIBRARY_OUTPUT_PATH EXECUTABLE_OUTPUT_PATH)

if(APPLE)
 add_definitions(-DDARWIN)
endif()



find_library(DL_LIB dl)
set (LIB_SRCS src/com_c.C src/assertion.C src/com_exception.C src/commpi.C src/COM_base.C src/DataItem.C src/Connectivity.C src/ComponentInterface.C src/Pane.C src/Element_accessors.C)
SET_SOURCE_FILES_PROPERTIES(${LIB_SRCS} src/com_f.C PROPERTIES COMPILE_FLAGS "${MPI_CXX_COMPILE_FLAGS}")
#set (LIB_SRCS src/com_c.C src/assertion.C src/com_exception.C src/commpi.C src/COM_substrate.C src/DataItem.C src/Connectivity.C src/ComponentInterface.C src/Pane.C src/Element_accessors.C src/ParallelAdapter.C)
set (FLIB_SRCS src/com_f.C src/m_pointers.f90 src/utilities.f90)
SET_SOURCE_FILES_PROPERTIES(src/m_pointers.f90 src/utilities.f90 PROPERTIES COMPILE_FLAGS "${MPI_Fortran_COMPILE_FLAGS}")
#set (TESTMOD_SRCS testing/src/TestModule.C)
#set (FTESTMOD_SRCS testing/src/FTestModule.f90)
#set (ALL_SRCS "${LIB_SRCS} ${FLIB_SRCS} ${FTESTMOD_SRCS} ${TESTMOD_SRCS}" )
set (ALL_SRCS "${LIB_SRCS} ${FLIB_SRCS}" )
set_source_files_properties(${ALL_SRCS} PROPERTIES COMPILE_FLAGS "-fPIC" )
#set(TEST_SRCS src/COMTest.C)

# rpath settings
SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

include_directories(include ${CMAKE_BINARY_DIR}/include)


IF(NOT BUILD_STATIC)
  add_library(SITCOM SHARED ${LIB_SRCS})
  add_library(SITCOMF SHARED ${FLIB_SRCS})
#  add_library(COMTESTMOD SHARED ${TESTMOD_SRCS})
#  add_library(COMFTESTMOD SHARED ${FTESTMOD_SRCS})
ELSE()
  add_library(SITCOM STATIC ${LIB_SRCS})
  add_library(SITCOMF STATIC ${FLIB_SRCS})
#  add_library(COMTESTMOD STATIC ${TESTMOD_SRCS})
#  add_library(COMFTESTMOD STATIC ${FTESTMOD_SRCS})
  add_definitions( -DSTATIC_LINK )
ENDIF()
target_link_libraries(SITCOM ${DL_LIB} ${MPI_CXX_LIBRARIES})
target_link_libraries(SITCOMF SITCOM ${MPI_Fortran_LIBRARIES})
SET_TARGET_PROPERTIES(SITCOM PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(SITCOMF PROPERTIES LINK_FLAGS "${MPI_Fortran_LINK_FLAGS}")



FILE(GLOB H_FILES include/*.H)
FILE(GLOB h_FILES include/*.h)
FILE(GLOB hpp_FILES include/*.hpp)
SET(COM_INC_FILES ${H_FILES} ${h_FILES} ${hpp_FILES})
 
#ADD_EXECUTABLE(com_test src/TestCOM.C)
#TARGET_LINK_LIBRARIES(com_test IRAD SITCOM)
#TARGET_LINK_LIBRARIES(COMTESTMOD SITCOM)
#TARGET_LINK_LIBRARIES(COMFTESTMOD SITCOMF)

#add_executable(ComTestDriver testing/util/TestModuleDriver.C)
#target_link_libraries(ComTestDriver SITCOM)

#add_executable(ComTestFDriver testing/util/TestModuleDriver.f90)
#set_target_properties(ComTestFDriver PROPERTIES LINKER_LANGUAGE Fortran)
#target_link_libraries(ComTestFDriver SITCOMF)

#add_subdirectory(Rocblas)
#add_subdirectory(Rocin)
#add_subdirectory(Rocout)
#add_subdirectory(Rocmap)
#add_subdirectory(Rocsurf)



#add_executable(testx ${TEST_SRCS})
#target_link_libraries(testx COM ${APPUTIL_LIB})

#ADD_TEST(RunAllTests ${EXECUTABLE_OUTPUT_PATH}/testx testresults.txt)
#ADD_TEST(TestObject:ValidHandle ${TEST_RESULTS} TestObject:ValidHandle testresults.txt)
#ADD_TEST(TestObject:Exists ${TEST_RESULTS} TestObject:Exists testresults.txt)
#ADD_TEST(RunCOMUnitTests ${CMAKE_BINARY_DIR}/bin/com_test -o com_testresults.txt)
#ADD_TEST(COM_ModuleLoadingTest:Runs ${CMAKE_BINARY_DIR}/bin/testresults COM_ModuleLoadingTest:Runs com_testresults.txt)
#ADD_TEST(COM_get_window:WorksForCModule ${CMAKE_BINARY_DIR}/bin/testresults COM_get_window:WorksForCModule com_testresults.txt)
#ADD_TEST(COM_get_window:WorksForFortranModule ${CMAKE_BINARY_DIR}/bin/testresults COM_get_window:WorksForFortranModule com_testresults.txt)

INSTALL(FILES ${COM_INC_FILES} DESTINATION include)
INSTALL(FILES ${CMAKE_BINARY_DIR}/include/FC.h DESTINATION include)
INSTALL(TARGETS SITCOM SITCOMF RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
