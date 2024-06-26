################################################################################
# Copy input files to build dir

add_custom_target(copy_doug_test_data ALL ${CMAKE_COMMAND} -E copy_directory
  ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_BINARY_DIR}/tuv_doug)

################################################################################
# library of cross section and quantum yield calculators based on Doug's
# original TUV code

add_library(tuv_doug)
target_sources(tuv_doug
  PRIVATE
    addpnt.f
    driver.F90
    gridck.f
    gridw.f
    gridz.f
    inter1.f
    inter2.f
    inter3.f
    inter4.f
    la_srb.f
    lymana.f
    rdo2xs.f
    schum.f
    ../test_utils.F90
)

add_subdirectory(JCALC)

target_link_libraries(tuv_doug PUBLIC musica::tuvx)

################################################################################

# Tests refactored configurations based on Doug's data sets and Fortran code
add_executable(test_data_sets data_sets.F90)
set_target_properties(test_data_sets PROPERTIES LINKER_LANGUAGE Fortran)
target_link_libraries(test_data_sets PUBLIC tuv_doug)
if(TUVX_ENABLE_OPENMP)
  target_link_libraries(test_data_sets PUBLIC OpenMP::OpenMP_Fortran)
endif()
add_tuvx_test(data_sets test_data_sets "" ${CMAKE_BINARY_DIR})

################################################################################

# Tests the Lymann-Alpha and Schumann Runge bands calculations in TUV-x against
# Doug's version used to generate the lookup tables used in CAM
add_executable(test_la_srb_lut test_la_srb.F90)
set_target_properties(test_la_srb_lut PROPERTIES LINKER_LANGUAGE Fortran)
target_link_libraries(test_la_srb_lut PUBLIC tuv_doug)
if(TUVX_ENABLE_OPENMP)
  target_link_libraries(test_la_srb_lut PUBLIC OpenMP::OpenMP_Fortran)
endif()
add_tuvx_test(la_srb_lut test_la_srb_lut "" ${CMAKE_BINARY_DIR})

################################################################################
