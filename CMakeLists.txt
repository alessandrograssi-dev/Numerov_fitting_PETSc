cmake_minimum_required(VERSION 3.10)
project(fit_numerov_PETSc LANGUAGES C Fortran)

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Fortran flags
set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -w -fno-automatic -fdefault-double-8 -fdefault-real-8 -g -ffpe-trap=denormal,invalid,zero,overflow -fcheck=all -finit-real=snan -ffpe-trap=invalid")

# PETSc setup
set(PETSC_INCLUDE_DIR "$ENV{HOME}/petsc-install/include")
set(PETSC_LIBRARY_DIR "$ENV{HOME}/petsc-install/lib")
file(GLOB PETSC_LIBS ${PETSC_LIBRARY_DIR}/*.so)

# Source files
file(GLOB SRC_F90
    src/libs/scattering_single_channel.f90
    src/libs/numerov.f90
    src/libs/spherical_bessel.f90
    src/libs/linear_and_quadratic_regression.f90
    src/libs/optimization.f90
    src/libs/potential_pw.f90
    src/libs/potentials/EFT_pless_new.f90
    src/libs/potentials/EFT_pless_fit.f90
)

file(GLOB SRC_F
    src/libs/potentials/av14.f
    src/libs/potentials/av18.f
    src/libs/potentials/EFT_pless.f
    src/libs/potentials/EFT_pless_utils.f
)

file(GLOB SRC_COMMON_C
    src/libs/read_input_file.c
)

# Executable
add_executable(fit_potential src/fit_potential.c ${SRC_F90} ${SRC_F} ${SRC_COMMON_C})
# Main program for 3PJ channel
add_executable(fit_potential_3PJ src/fit_potential_3PJ.c ${SRC_F90} ${SRC_F} ${SRC_COMMON_C})

# Include directories
target_include_directories(fit_potential PRIVATE ${PETSC_INCLUDE_DIR})
target_include_directories(fit_potential_3PJ PRIVATE ${PETSC_INCLUDE_DIR})

# Link libraries
find_package(GSL REQUIRED)
find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)
target_link_libraries(fit_potential PRIVATE ${PETSC_LIBS} GSL::gsl GSL::gslcblas ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES})
target_link_libraries(fit_potential_3PJ PRIVATE ${PETSC_LIBS} GSL::gsl GSL::gslcblas ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES})

# Fortran runtime linking if needed
enable_language(Fortran)
include(CheckFortranFunctionExists)
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${CMAKE_Fortran_FLAGS}")