find_package(Doxygen)

if(DOXYGEN_FOUND)
  set(PARC_REMOTE http://githubenterprise.parc.com/CCNX/LongBow)
  set(GITHUB_REMOTE http://github.com/PARC/LongBow)
  configure_file(longbow.doxygen.in config.doxygen @ONLY)

  install(DIRECTORY images/            DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/images)
  install(DIRECTORY doxygen-extras/    DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/doxygen-extras)
  install(DIRECTORY examples/          DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/examples)
  install(FILES DoxygenLayout.xml      DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

  set(HTML_PATH  ${CMAKE_CURRENT_BINARY_DIR}/generated-documentation/html)

  add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/documentation.tar
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/config.doxygen
        COMMAND tar cf documentation.tar -C ${HTML_PATH} .
        COMMENT "The file documentation.tar contains the HTML website suitable to update/replace the gh-pages branch of this repository." VERBATIM )

  add_custom_target(documentation
	DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/documentation.tar
        WORKING_DIRECTORY ${HTML_PATH})

  add_custom_command(OUTPUT ${HTML_PATH}/.git/config
	DEPENDS documentation
        WORKING_DIRECTORY ${HTML_PATH}
	COMMAND git init .
	COMMAND git checkout -B gh-pages
	COMMAND git add .
	COMMAND git commit -am 'Override all previous versions'
	COMMAND git remote add parc ${PARC_REMOTE}
	COMMAND git remote add github ${GITHUB_REMOTE})

  add_custom_target(documentation-install-parc
	DEPENDS ${HTML_PATH}/.git/config
        WORKING_DIRECTORY ${HTML_PATH}
	COMMAND git push parc gh-pages -f)

  add_custom_target(documentation-install-github
	DEPENDS ${HTML_PATH}/.git/config
        WORKING_DIRECTORY ${HTML_PATH}
	COMMAND git push github gh-pages -f)

endif(DOXYGEN_FOUND)
