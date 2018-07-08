add_subdirectory(sandbox_shared_lib)

add_executable(sandbox sandbox.cpp)
target_link_libraries( sandbox cereal::cereal )

add_executable(sandbox_json sandbox_json.cpp)
target_link_libraries( sandbox_json cereal::cereal )

add_executable(sandbox_rtti sandbox_rtti.cpp)
target_link_libraries( sandbox_rtti cereal::cereal )

add_executable(sandbox_vs sandbox_vs.cpp)
target_link_libraries(sandbox_vs sandbox_vs_dll)
include_directories(sandbox_shared_lib)

if(Boost_FOUND)
  add_executable(performance performance.cpp)
  if(MSVC)
    set_target_properties(performance PROPERTIES COMPILE_DEFINITIONS "BOOST_SERIALIZATION_DYN_LINK")
  endif()
  target_link_libraries(performance cereal::cereal ${Boost_LIBRARIES})
endif(Boost_FOUND)
