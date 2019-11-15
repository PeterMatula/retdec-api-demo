
add_executable(demo demo.cpp)
target_link_libraries(demo retdec-api)
install(TARGETS demo RUNTIME DESTINATION bin)
