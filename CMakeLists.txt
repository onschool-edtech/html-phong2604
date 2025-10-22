cmake_minimum_required(VERSION 3.10)
project(MyApp CXX)

add_executable(MyApp main.cpp)
enable_testing()
add_test(NAME RunMyApp COMMAND MyApp)