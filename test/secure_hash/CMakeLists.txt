add_executable (test_secure_hash test_secure_hash.F90)
target_link_libraries(test_secure_hash petaca)
add_test(secure_hash test_secure_hash)

#add_executable (time_hash time_hash.F90)
#target_link_libraries(time_hash petaca)
