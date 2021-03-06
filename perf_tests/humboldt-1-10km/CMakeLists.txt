# Copy files to build directory
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/input_albany_PopulateMesh_Wedge.yaml
               ${CMAKE_CURRENT_BINARY_DIR}/input_albany_PopulateMesh_Wedge.yaml COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/input_albany_Coupled_Ifpack2_Wedge.yaml
               ${CMAKE_CURRENT_BINARY_DIR}/input_albany_Coupled_Ifpack2_Wedge.yaml COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/input_albany_Coupled_FROSch_Wedge.yaml
               ${CMAKE_CURRENT_BINARY_DIR}/input_albany_Coupled_FROSch_Wedge.yaml COPYONLY)

# Create symlink for mesh directory
SET(MESH_PATH "${MESH_FILE_DIR}/humboldt-1-10km")
execute_process(COMMAND ${CMAKE_COMMAND} -E create_symlink
                ${MESH_PATH} ${CMAKE_CURRENT_BINARY_DIR}/mesh)

# Decompose the mesh
execute_process(COMMAND ${ALBANY_SEACAS_DECOMP} --processors ${ALBANY_NUM_PROCS} --root ${CMAKE_CURRENT_BINARY_DIR}/ --subdir mesh-decomp
                ${MESH_PATH}/humboldt_2d.exo)

# Name the test with the directory name
get_filename_component(testName ${CMAKE_CURRENT_SOURCE_DIR} NAME)

# Add tests to populate mesh with ascii inputs
add_test(${testName}_pop_wdg_np${ALBANY_NUM_PROCS} ${ALBANY_SFAD24_EXE} input_albany_PopulateMesh_Wedge.yaml ${ALBANY_EXTRA_ARGS})

# Add tests for performance
add_test(${testName}_cop_if2_wdg_np${ALBANY_NUM_PROCS} ${ALBANY_SFAD24_EXE} input_albany_Coupled_Ifpack2_Wedge.yaml ${ALBANY_EXTRA_ARGS})
add_test(${testName}_cop_fro_wdg_np${ALBANY_NUM_PROCS} ${ALBANY_SFAD24_EXE} input_albany_Coupled_FROSch_Wedge.yaml ${ALBANY_EXTRA_ARGS})

# Add dependencies
set_tests_properties(${testName}_pop_wdg_np${ALBANY_NUM_PROCS}
                     PROPERTIES FIXTURES_SETUP ${testName}_pop_wdg)
set_tests_properties(${testName}_cop_if2_wdg_np${ALBANY_NUM_PROCS}
                     ${testName}_cop_fro_wdg_np${ALBANY_NUM_PROCS}
                     PROPERTIES FIXTURES_REQUIRED ${testName}_pop_wdg)

