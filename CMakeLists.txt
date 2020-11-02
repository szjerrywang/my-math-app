cmake_minimum_required(VERSION 3.10)

# set the project name
project(MyMathApp VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(USE_MYMATH "Use tutorial provided math implementation" ON)

configure_file(MyMathAppConfig.h.in MyMathAppConfig.h)

if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)
    # list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()


# add the executable
add_executable(MyMathApp mymathapp.cpp)

target_link_libraries(MyMathApp PUBLIC ${EXTRA_LIBS})

target_include_directories(MyMathApp PUBLIC
"${PROJECT_BINARY_DIR}"
# ${EXTRA_INCLUDES}        
)
include(CheckSymbolExists)
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
if(NOT (HAVE_LOG AND HAVE_EXP))
  unset(HAVE_LOG CACHE)
  unset(HAVE_EXP CACHE)
  set(CMAKE_REQUIRED_LIBRARIES "m")
  check_symbol_exists(log "math.h" HAVE_LOG)
  check_symbol_exists(exp "math.h" HAVE_EXP)
  if(HAVE_LOG AND HAVE_EXP)
    target_link_libraries(MathFunctions PRIVATE m)
  endif()
endif()

install(TARGETS MyMathApp DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/MyMathAppConfig.h" DESTINATION include)

enable_testing()

# does the application run
add_test(NAME Runs COMMAND MyMathApp 25)

# does the usage message work?
add_test(NAME Usage COMMAND MyMathApp)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(MyMathApp 4 "4 is 2")
do_test(MyMathApp 9 "9 is 3")
do_test(MyMathApp 5 "5 is 2.236")
do_test(MyMathApp 7 "7 is 2.645")
do_test(MyMathApp 25 "25 is 5")
do_test(MyMathApp -25 "-25 is [-nan|nan|0]")
do_test(MyMathApp 0.0001 "0.0001 is 0.01")

