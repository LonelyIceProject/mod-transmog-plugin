# Builds mod-transmog as a plugin, like a package spec: the module's own repository ("source" in plugin.json) at the
# pinned commit, with patches/*.patch applied, and plugin/plugin.cpp as the entry point. After changing a patch,
# delete _deps/mod-transmog-* in the build folder so the checkout is patched again.
include(FetchContent)
find_package(Git REQUIRED)

file(READ "${CMAKE_CURRENT_SOURCE_DIR}/plugin.json" manifest)
string(JSON id GET "${manifest}" id)
string(JSON repo GET "${manifest}" source repo)
string(JSON commit GET "${manifest}" source commit)
file(GLOB patches "${CMAKE_CURRENT_SOURCE_DIR}/patches/*.patch")
set(patch_command "")
if (patches)
  # Runs again whenever the module is fetched anew or reconfigured, so it starts from the pinned commit every time.
  set(patch_command PATCH_COMMAND "${GIT_EXECUTABLE}" reset --quiet --hard COMMAND "${GIT_EXECUTABLE}" clean --quiet -fd
    COMMAND "${GIT_EXECUTABLE}" apply --whitespace=nowarn ${patches})
endif()
# SOURCE_SUBDIR names no folder: the module is only fetched, its own build files are not used.
FetchContent_Declare(mod-transmog GIT_REPOSITORY "${repo}" GIT_TAG "${commit}" SOURCE_SUBDIR none ${patch_command})
FetchContent_MakeAvailable(mod-transmog)
set(module "${mod-transmog_SOURCE_DIR}")

file(GLOB_RECURSE sources "${module}/src/*.cpp" "${module}/src/*.h")
AddPlugin(mod-transmog SOURCES ${sources} plugin/plugin.cpp)
if (NOT TARGET mod-transmog)
  return()
endif()
target_include_directories(mod-transmog PRIVATE "${module}/src")

# The module's conf and sql folders, laid out after the plugin's own files.
set(copy "")
foreach(sub conf data)
  if (EXISTS "${module}/${sub}")
    list(APPEND copy COMMAND ${CMAKE_COMMAND} -E copy_directory "${module}/${sub}" "${AC_PLUGINS_OUTPUT_DIR}/${id}/${sub}")
  endif()
endforeach()
add_custom_target(mod-transmog-module-files ALL ${copy} VERBATIM)
set_target_properties(mod-transmog-module-files PROPERTIES FOLDER "plugins")
add_dependencies(mod-transmog-module-files mod-transmog-files)
add_dependencies(mod-transmog mod-transmog-module-files)
