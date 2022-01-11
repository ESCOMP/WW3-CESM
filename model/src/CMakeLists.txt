cmake_minimum_required(VERSION 3.10)
project(wav Fortran)

message("Using CIME in ${CIMEROOT} with compiler ${COMPILER}")
include(${CASEROOT}/Macros.cmake)
if (MPILIB STREQUAL mpi-serial)
  set(CMAKE_Fortran_COMPILER ${SFC})
  set(CMAKE_C_COMPILER ${SCC})
else()
  set(CMAKE_Fortran_COMPILER ${MPIFC})
  set(CMAKE_C_COMPILER ${MPICC})
endif()
set(CMAKE_Fortran_FLAGS "${FFLAGS} ${CPPDEFS} -I${LIBROOT}/include -I${LIBROOT}/finclude -I${LIBROOT}/nuopc/esmf/${NINST_VALUE}/include")
add_compile_definitions(CESMCOUPLED)

# Set ESMF_F90COMPILEPATHS
# convert esmf.mk makefile variables to cmake variables until ESMF
# provides proper cmake package
if (DEFINED ENV{ESMFMKFILE})
  message("ESMFMKFILE:   $ENV{ESMFMKFILE}")
else()
  message(FATAL_ERROR "ESMFMKFILE env variable is not defined")
endif()
set(ESMFMKFILE $ENV{ESMFMKFILE})
file(STRINGS ${ESMFMKFILE} esmf_mk_text)
foreach(line ${esmf_mk_text})
  string(REGEX REPLACE "^[ ]+" "" line ${line}) # strip leading spaces
  if (line MATCHES "^ESMF_*")                   # process only line starting with ESMF_
    string(REGEX MATCH "^ESMF_[^=]+" esmf_name ${line})
    string(REPLACE "${esmf_name}=" "" emsf_value ${line})
    set(${esmf_name} "${emsf_value}")
  endif()
endforeach()
string(REPLACE "-I" "" ESMF_F90COMPILEPATHS ${ESMF_F90COMPILEPATHS})
string(REPLACE " " ";" ESMF_F90COMPILEPATHS ${ESMF_F90COMPILEPATHS})
message("ESMF_F90COMPILEPATHS:   ${ESMF_F90COMPILEPATHS}")

# Set source files
set(SRCFILES wav_comp_nuopc.F90
             wav_import_export.F90
             wav_kind_mod.F90
             wav_shel_inp.F90
             wav_shr_mod.F90
             constants.F90
             w3adatmd.F90
             w3arrymd.F90
             w3cspcmd.F90
             w3dispmd.F90
             w3fldsmd.F90
             w3flx1md.F90
             w3gdatmd.F90
             w3gsrumd.F90
             w3idatmd.F90
             w3initmd.F90
             w3iobcmd.F90
             w3iogomd.F90
             w3iogoncdmd.F90
             w3iogrmd.F90
             w3iopomd.F90
             w3iorsmd.F90
             w3iosfmd.F90
             w3iotrmd.F90
             w3macros.h
             w3odatmd.F90
             w3parall.F90
             w3partmd.F90
             w3pro3md.F90
             w3profsmd.F90
             w3sbt1md.F90
             w3sdb1md.F90
             w3servmd.F90
             w3sic4md.F90
             w3sln1md.F90
             w3snl1md.F90
             w3src4md.F90
             w3srcemd.F90
             w3timemd.F90
             w3triamd.F90
             w3updtmd.F90
             w3uqckmd.F90
             w3wavemd.F90
             w3wdasmd.F90
             w3wdatmd.F90
             wmmdatmd.F90)

foreach(FILE ${SRCFILES})
  if(EXISTS "${CASEROOT}/SourceMods/src.ww3/${FILE}")
    list(REMOVE_ITEM SRCFILES ${FILE})
    list(APPEND SRCFILES "${CASEROOT}/SourceMods/src.dww3/${FILE}")
    message("Using ${FILE} from ${CASEROOT}/SourceMods/src.ww3")
  endif()
endforeach()

message("CMAKE_CURRENT_BINARY_DIR is ${CMAKE_CURRENT_BINARY_DIR}")
message("PROJECT_BINARY_DIR is ${PROJECT_BINARY_DIR}")

ADD_DEFINITIONS(-DCESMCOUPLED)

add_library(wav ${SRCFILES})
target_include_directories (wav PRIVATE ${ESMF_F90COMPILEPATHS})

install(TARGETS wav)
