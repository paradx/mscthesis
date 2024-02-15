CMAKE_MINIMUM_REQUIRED(VERSION 3.11)
PROJECT(LatexProject NONE)

FIND_PACKAGE(LATEX)
IF(LATEX_FOUND)
  IF(PDFLATEX_FOUND)
		MESSAGE(STATUS "pdflatex found - ${PDFLATEX_COMPILER}")
	ENDIF()
  IF(BIBER_COMPILER)
		MESSAGE(STATUS "biber found - ${BIBER_COMPILER}")
	ENDIF()
ELSE()
	MESSAGE(ERROR "No latex tools found!")
ENDIF()


# The directory where the final build data will be stored.
SET(LATEX_OUTPUT_PATH build)
SET(OUT_DIRECTORY "${CMAKE_SOURCE_DIR}/${LATEX_OUTPUT_PATH}")
# The source file containing the reference for cites.
SET(BIB_REFERENCE_PATH ${CMAKE_SOURCE_DIR}/src/refs.bib)
configure_file(${BIB_REFERENCE_PATH} ${OUT_DIRECTORY} @ONLY)
# The source file containing the abbreviations
SET(ABBREV_REF_PART ${CMAKE_SOURCE_DIR}/src/abbrvs.bib)
configure_file(${ABBREV_REF_PART} ${OUT_DIRECTORY} @ONLY)

# Latex source file.
SET( MAIN_TEX_BASE_FILENAME "thesis")
SET( MAIN_TEX "${CMAKE_SOURCE_DIR}/src/${MAIN_TEX_BASE_FILENAME}.tex")
SET( MAIN_IDX "${LATEX_OUTPUT_PATH}/${MAIN_TEX_BASE_FILENAME}.idx")
SET( MAIN_AUX "${LATEX_OUTPUT_PATH}/${MAIN_TEX_BASE_FILENAME}.aux")

# Directory where the source files are located.
SET( WORKINGDIR "${CMAKE_SOURCE_DIR}/src" )

# First pass.
# enabled shell-escape since I use the SVG pkg
ADD_CUSTOM_TARGET( latex-prebuild
		COMMAND ${PDFLATEX_COMPILER} -shell-escape -output-directory ${OUT_DIRECTORY} -draftmode -interaction=nonstopmode ${MAIN_TEX}
		COMMAND ${PDFLATEX_COMPILER} -shell-escape -output-directory ${OUT_DIRECTORY} -draftmode -interaction=nonstopmode ${MAIN_TEX}
		COMMENT "Starting Prebuild."
		WORKING_DIRECTORY ${WORKINGDIR}
		DEPENDS ${MAIN_TEX})

# Generate what citation found in the latex file.
ADD_CUSTOM_TARGET( latex-bibreferences
			COMMAND ${BIBER_COMPILER} build/thesis
			COMMENT "Read and create main bib references file."
			DEPENDS  ${CMAKE_SOURCE_DIR}/build/thesis.aux
			DEPENDS  ${CMAKE_SOURCE_DIR}/src/refs.bib
    )
ADD_DEPENDENCIES( latex-bibreferences latex-prebuild)

# Generate the glossary 
ADD_CUSTOM_TARGET( latex-glossaries
      COMMAND makeglossaries thesis 
      WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build
			COMMENT "Create glossary."
			DEPENDS  ${CMAKE_SOURCE_DIR}/build/thesis.gls
			DEPENDS  ${CMAKE_SOURCE_DIR}/build/thesis.xdy
    )
ADD_DEPENDENCIES( latex-glossaries latex-prebuild)


# Second pass - generate the final pdf.
ADD_CUSTOM_TARGET( latex-pdf	
			COMMAND ${PDFLATEX_COMPILER} -output-directory ${OUT_DIRECTORY} ${MAIN_TEX} 
			WORKING_DIRECTORY ${WORKINGDIR}
			COMMENT "Assembling the final pdf file."
			DEPENDS  ${MAIN_TEX})
ADD_DEPENDENCIES( latex-pdf latex-prebuild latex-bibreferences latex-glossaries)

ADD_CUSTOM_TARGET(all-formats ALL) # Entry point of execution.
ADD_DEPENDENCIES( all-formats latex-pdf)
