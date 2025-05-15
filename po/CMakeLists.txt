# Make a function to process & install the internationalised .desktop files
function(install_internationalized_desktop_file FILE DESTINATION)
    get_filename_component(fileName ${FILE} NAME)

    # Define a custom subdirectory within the current binary directory for these outputs
    set(outputDir ${CMAKE_CURRENT_BINARY_DIR}/internationalizeddesktopfile)
    set(outputFile ${outputDir}/${fileName})

    # Determine the unambiguous, full path to the source template FILE.
    # This is crucial for the --template argument in the COMMAND.
    if(IS_ABSOLUTE ${FILE})
        set(abs_template_path ${FILE})
    else()
        # Resolve FILE relative to the current source directory
        set(abs_template_path ${CMAKE_CURRENT_SOURCE_DIR}/${FILE})
    endif()

    # Safety check: This condition is now less likely to be true even in in-source builds
    # due to the 'outputDir', unless FILE itself points into that specific subdirectory.
    if("${abs_template_path}" STREQUAL "${outputFile}")
        message(FATAL_ERROR "Input template file '${abs_template_path}' and output file '${outputFile}' are the same path. This will cause a dependency cycle. Please ensure the input FILE points to a source template, not the build output location.")
    endif()

    # The build system (Ninja, Make, etc.) will typically create the outputDir
    # when the custom command is executed to produce ${outputFile}.

    add_custom_command(
        OUTPUT ${outputFile}
        # Use the unambiguous abs_template_path for --template.
        # The -d path for .po files is kept as CMAKE_BINARY_DIR from your original.
        COMMAND ${GETTEXT_MSGFMT_EXECUTABLE} --desktop --template=${abs_template_path} -d ${CMAKE_BINARY_DIR}/po/desktopfiles -o ${outputFile}
        # The custom command depends on the source template file.
        DEPENDS ${abs_template_path}
        # VERBATIM is good practice for custom commands.
        VERBATIM
        # Adding a comment helps in build logs.
        COMMENT "Generating internationalized ${fileName} into ${outputDir} from ${abs_template_path}"
    )

    # Create a unique name for the custom target to avoid redefinition.
    # The ALL keyword makes this target part of the default build.
    add_custom_target(process_desktop_file_${fileName} ALL DEPENDS ${outputFile})

    install(PROGRAMS ${outputFile} DESTINATION ${DESTINATION})
endfunction()
