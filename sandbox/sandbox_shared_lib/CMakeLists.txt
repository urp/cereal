add_library(sandbox_vs_dll SHARED base.cpp derived.cpp)
target_link_libraries( sandbox_vs_dll cereal::cereal )
