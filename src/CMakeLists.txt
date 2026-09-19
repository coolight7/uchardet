set(
	UCHARDET_HEADERS
	uchardet.h
)

set(
	UCHARDET_SOURCES
	CharDistribution.cpp
	JpCntx.cpp
	LangModels/LangArabicModel.cpp
	LangModels/LangBelarusianModel.cpp
	LangModels/LangBulgarianModel.cpp
	LangModels/LangCatalanModel.cpp
    LangModels/LangCroatianModel.cpp
    LangModels/LangCzechModel.cpp
    LangModels/LangEnglishModel.cpp
	LangModels/LangEsperantoModel.cpp
	LangModels/LangEstonianModel.cpp
	LangModels/LangFinnishModel.cpp
	LangModels/LangFrenchModel.cpp
	LangModels/LangDanishModel.cpp
	LangModels/LangGermanModel.cpp
	LangModels/LangGeorgianModel.cpp
	LangModels/LangGreekModel.cpp
	LangModels/LangHungarianModel.cpp
	LangModels/LangHebrewModel.cpp
	LangModels/LangHindiModel.cpp
	LangModels/LangIrishModel.cpp
	LangModels/LangItalianModel.cpp
    LangModels/LangLithuanianModel.cpp
    LangModels/LangLatvianModel.cpp
    LangModels/LangMacedonianModel.cpp
    LangModels/LangMalteseModel.cpp
    LangModels/LangPolishModel.cpp
    LangModels/LangPortugueseModel.cpp
	LangModels/LangRomanianModel.cpp
	LangModels/LangRussianModel.cpp
	LangModels/LangSerbianModel.cpp
	LangModels/LangSlovakModel.cpp
    LangModels/LangSloveneModel.cpp
    LangModels/LangSwedishModel.cpp
	LangModels/LangSpanishModel.cpp
	LangModels/LangThaiModel.cpp
	LangModels/LangTurkishModel.cpp
	LangModels/LangUkrainianModel.cpp
	LangModels/LangVietnameseModel.cpp
	LangModels/LangNorwegianModel.cpp
	nsHebrewProber.cpp
	nsCharSetProber.cpp
	nsBig5Prober.cpp
	nsEUCJPProber.cpp
	nsEUCKRProber.cpp
	nsEUCTWProber.cpp
	nsEscCharsetProber.cpp
	nsEscSM.cpp
	nsGB2312Prober.cpp
	nsJohabProber.cpp
	nsMBCSGroupProber.cpp
	nsMBCSSM.cpp
	nsSBCSGroupProber.cpp
	nsSBCharSetProber.cpp
	nsSJISProber.cpp
	nsUTF8Prober.cpp
	nsLanguageDetector.cpp
	nsCJKDetector.cpp
	nsLatin1Prober.cpp
	nsUniversalDetector.cpp
	uchardet.cpp
)

set (UCHARDET_LIBRARY libuchardet)
set (UCHARDET_LIBRARY libuchardet PARENT_SCOPE)

if (BUILD_STATIC AND BUILD_SHARED_LIBS)
	set (UCHARDET_STATIC_LIBRARY libuchardet_static)
endif ()

# 编译参数按编译器区分: -Wall / -O0 / -g3 都是 GCC/Clang 风格参数。
# MSVC 下 -Wall 会被 CMake 映射为 /Wall (EnableAllWarnings), 于是 C4711 (自动内联
# 提示)、C4710 (未内联提示)、C5045 (Spectre 缓解提示) 这类信息类告警全部输出,
# 编译日志噪音很大; -O0 / -g3 则直接被 cl 判为未知参数 (warning D9002)。因此这些
# 参数只对非 MSVC 编译器启用。
if (MSVC)
	add_definitions(
		-DVERSION="${UCHARDET_VERSION}"
	)
else ()
	add_definitions(
		-DVERSION="${UCHARDET_VERSION}"
		-Wall
	)

	if (CMAKE_BUILD_TYPE MATCHES Debug)
		add_definitions(
			-O0
			-g3
		)
	endif (CMAKE_BUILD_TYPE MATCHES Debug)
endif ()

add_library(
	${UCHARDET_LIBRARY}
	${UCHARDET_SOURCES}
)
target_compile_definitions("${UCHARDET_LIBRARY}" PRIVATE BUILDING_UCHARDET)
if(BUILD_SHARED_LIBS)
	target_compile_definitions("${UCHARDET_LIBRARY}" PUBLIC UCHARDET_SHARED)
endif()

target_include_directories(${UCHARDET_LIBRARY}
	PUBLIC
		"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
		"$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PACKAGE_NAME}>"
)

if (UCHARDET_STATIC_LIBRARY)
	add_library(
		${UCHARDET_STATIC_LIBRARY}
		STATIC
		${UCHARDET_SOURCES}
	)
	target_compile_definitions("${UCHARDET_STATIC_LIBRARY}" PRIVATE BUILDING_UCHARDET)

	target_include_directories(${UCHARDET_STATIC_LIBRARY}
		PUBLIC
			"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
			"$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PACKAGE_NAME}>"
	)
endif (UCHARDET_STATIC_LIBRARY)

set_target_properties(
	${UCHARDET_LIBRARY}
	PROPERTIES
		LINKER_LANGUAGE
			CXX
		OUTPUT_NAME
			${PACKAGE_NAME}
		VERSION
			${UCHARDET_VERSION}
		SOVERSION
			${UCHARDET_VERSION_MAJOR}
)

if (UCHARDET_STATIC_LIBRARY)
	set_target_properties(
		${UCHARDET_STATIC_LIBRARY}
		PROPERTIES
			LINKER_LANGUAGE
				CXX
			OUTPUT_NAME
				${PACKAGE_NAME}
	)
endif (UCHARDET_STATIC_LIBRARY)

if (NOT WIN32)
	install(
		TARGETS
			${UCHARDET_LIBRARY}
		EXPORT
			UchardetTargets
		LIBRARY DESTINATION
			${CMAKE_INSTALL_LIBDIR}
		ARCHIVE DESTINATION
			${CMAKE_INSTALL_LIBDIR}
	)
else (NOT WIN32)
	install(
		TARGETS
			${UCHARDET_LIBRARY}
		EXPORT
			UchardetTargets
		RUNTIME DESTINATION
			${CMAKE_INSTALL_BINDIR}
		ARCHIVE DESTINATION
			${CMAKE_INSTALL_LIBDIR}
	)
endif (NOT WIN32)

if (UCHARDET_STATIC_LIBRARY)
	install(
		TARGETS
			${UCHARDET_STATIC_LIBRARY}
		EXPORT
			UchardetTargets
		ARCHIVE DESTINATION
			${CMAKE_INSTALL_LIBDIR}
	)
endif (UCHARDET_STATIC_LIBRARY)

install(
	FILES
		${UCHARDET_HEADERS}
	DESTINATION
		${CMAKE_INSTALL_INCLUDEDIR}/${PACKAGE_NAME}
)

include(symbols.cmake)

if (BUILD_BINARY)
	add_subdirectory(tools)
endif (BUILD_BINARY)
