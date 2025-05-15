# Make a function to process & install the internationalised .desktop files in the base dir
function(install_internationalized_desktop_file FILE DESTINATION)
    get_filename_component(fileName ${FILE} NAME)

    # Output file path (kept CMAKE_BINARY_DIR as per your original, adjust if needed)
    # For outputs generated from a specific CMakeLists.txt, CMAKE_CURRENT_BINARY_DIR is often preferred.
    set(outputFile ${CMAKE_BINARY_DIR}/${fileName})

    # Determine the unambiguous, full path to the source template FILE.
    # This is crucial for the --template argument in the COMMAND.
    if(IS_ABSOLUTE ${FILE})
        set(abs_template_path ${FILE})
    else()
        # Resolve FILE relative to the current source directory
        set(abs_template_path ${CMAKE_CURRENT_SOURCE_DIR}/${FILE})
    endif()

    # Safety check: if, after resolution, the input template path is identical
    # to the output path, it's a guaranteed cycle (common in in-source builds).
    if("${abs_template_path}" STREQUAL "${outputFile}")
        message(FATAL_ERROR "Input template file '${abs_template_path}' and output file '${outputFile}' are the same path. This will cause a dependency cycle. Please ensure you are using an out-of-source build and the input FILE points to a source template, not the build output location.")
    endif()

    add_custom_command(
        OUTPUT ${outputFile}
        # Use the unambiguous abs_template_path for --template.
        # The -d path for .po files is kept as CMAKE_BINARY_DIR from your original.
        COMMAND ${GETTEXT_MSGFMT_EXECUTABLE} --desktop --template=${abs_template_path} -d ${CMAKE_BINARY_DIR}/po/desktopfiles -o ${outputFile}
        # The custom command depends on the source template file.
        # You should also add dependencies for the actual .po files if they can change.
        DEPENDS ${abs_template_path}
        # VERBATIM is good practice for custom commands to ensure arguments are passed exactly.
        VERBATIM
        # Adding a comment helps in build logs.
        COMMENT "Generating internationalized ${fileName} from ${abs_template_path}"
    )

    # Create a unique name for the custom target to avoid redefinition if the function is called multiple times.
    # The ALL keyword makes this target part of the default build (i.e., 'ninja' or 'make' will build it).
    add_custom_target(process_desktop_file_${fileName} ALL DEPENDS ${outputFile})

    install(PROGRAMS ${outputFile} DESTINATION ${DESTINATION})
endfunction()
