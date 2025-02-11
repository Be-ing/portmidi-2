# pm_java

if(BUILD_PMDEFAULTS)
  set(JPORTMIDICLASS JPortMidi.class JPortMidiException.class  
                     JPortMidiApi.class)  
  set(PMDEFAULTSCLASS PmDefaultsFrame.class PmDefaults.class)  
  prepend_path(JPORTMIDIPATH jportmidi/ ${JPORTMIDICLASS}) 
  prepend_path(PMDEFAULTSPATH pmdefaults/ ${PMDEFAULTSCLASS}) 
  set(PMDEFAULTS_ALL ${JPORTMIDIPATH} ${PMDEFAULTSPATH})

  # SOME DEBUGGING INFORMATION IS COMMENTED OUT HERE...
  #message(STATUS "Where does CMAKE put things?..."
  #message(STATUS "  LIBRARY_OUTPUT_DIRECTORY ${LIBRARY_OUTPUT_DIRECTORY}") 
  #message(STATUS "  LIBRARY_OUTPUT_PATH ${LIBRARY_OUTPUT_PATH}") 
  #message(STATUS "  RUNTIME_OUTPUT_DIRECTORY ${RUNTIME_OUTPUT_DIRECTORY}") 
  #message(STATUS
  #   "  CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}") 
  #message(STATUS
  #   "  CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}") 
  #message(STATUS "  CMAKE_CURRENT_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}") 
  #message(STATUS 
  #   "  CMAKE_BUILD_FILES_DIRECTORY ${CMAKE_BUILD_FILES_DIRECTORY}") 
  #message(STATUS "  CMAKE_BUILD_DIRECTORY ${CMAKE_BUILD_DIRECTORY}") 
  #message(STATUS "  CMAKE_BINARY_DIR ${CMAKE_BINARY_DIR}") 
  #message(STATUS "  EXECUTABLE_OUTPUT_PATH ${EXECUTABLE_OUTPUT_PATH}") 

  add_custom_command(OUTPUT pmdefaults/PmDefaultsFrame.class 
          COMMAND javac -classpath . pmdefaults/PmDefaultsFrame.java 
          MAIN_DEPENDENCY pmdefaults/PmDefaultsFrame.java 
          DEPENDS pmdefaults/PmDefaults.java 
          WORKING_DIRECTORY .) 
  add_custom_command(OUTPUT pmdefaults/PmDefaults.class 
          COMMAND javac -classpath . pmdefaults/PmDefaults.java 
          MAIN_DEPENDENCY pmdefaults/PmDefaults.java 
          DEPENDS pmdefaults/PmDefaultsFrame.java 
          WORKING_DIRECTORY .) 
  add_custom_target(pmdefaults ALL 
          DEPENDS ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/pmdefaults.jar) 
  if(UNIX)
    add_custom_command(OUTPUT ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/pmdefaults.jar 
                              ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/pmdefaults
          COMMAND cp pmdefaults/portmusic_logo.png . 
          COMMAND jar cmf pmdefaults/manifest.txt pmdefaults.jar 
                  pmdefaults/*.class portmusic_logo.png jportmidi/*.class 
          COMMAND mv pmdefaults.jar ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
          COMMAND rm portmusic_logo.png 
          # create a command to run pmdefaults:
          COMMAND cp pmdefaults/pmdefaults ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
          COMMAND chmod +x ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/pmdefaults
          MAIN_DEPENDENCY pmdefaults/PmDefaults.class 
          DEPENDS ${PMDEFAULTS_ALL}
          WORKING_DIRECTORY .) 
    if(BUILD_JAVA_NATIVE_INTERFACE) 
      include(FindJNI) 
      # message(STATUS "JAVA_JVM_LIB_PATH is " ${JAVA_JVM_LIB_PATH}) 
      # message(STATUS "JAVA_INCLUDE_PATH is " ${JAVA_INCLUDE_PATH}) 
      # note: should use JAVA_JVM_LIB_PATH, but it is not set properly 
      # note: user might need to set JAVA_INCLUDE_PATH manually 
      #
      # this will probably break on BSD and other Unix systems; the fix 
      # depends on whether FindJNI can find Java or not. If yes, then 
      # we should try to rely on automatically set JAVA_INCLUDE_PATH and 
      # JAVA_INCLUDE_PATH2; if no, then we need to make both JAVA_INCLUDE_PATH 
      # and JAVA_INCLUDE_PATH2 set by user (will need clear documentation 
      # because JAVA_INCLUDE_PATH2 is pretty obscure) 
      set(JAVA_INCLUDE_PATH  ${JAVA_INCLUDE_PATH-UNKNOWN}
          CACHE STRING "where to find Java SDK include directory") 
      set(JAVA_INCLUDE_PATHS ${JAVA_INCLUDE_PATH} ${JAVA_INCLUDE_PATH}/linux) 
      # libjvm.so is found relative to JAVA_INCLUDE_PATH: 
      set(JAVAVM_LIB ${JAVA_INCLUDE_PATH}/../jre/lib/i386/client/libjvm.so) 
    endif(BUILD_JAVA_NATIVE_INTERFACE) 
    if(APPLE)
    else(APPLE)
      # install the libraries (Linux only)
      INSTALL(FILES ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/pmdefaults.jar
        DESTINATION /usr/share/java)
      INSTALL(PROGRAMS ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/pmdefaults
        DESTINATION /usr/local/bin)
    endif(APPLE)
  else(UNIX)
    if(WIN32) 
      add_custom_command(OUTPUT ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/pmdefaults.jar
          COMMAND copy pmdefaults\\portmusic_logo.png . 
          COMMAND echo Running jar to build pmdefaults.jar      
          COMMAND jar cmf pmdefaults/manifest.txt pmdefaults.jar 
                  pmdefaults/*.class portmusic_logo.png jportmidi/*.class 
          COMMAND del portmusic_logo.png 
          MAIN_DEPENDENCY pmdefaults/PmDefaults.class 
          DEPENDS ${PMDEFAULTS_ALL}
          WORKING_DIRECTORY .) 
      # /MDd is multithread debug DLL, /MTd is multithread debug 
      # /MD is multithread DLL, /MT is multithread 

      if(BUILD_JAVA_NATIVE_INTERFACE) 
        include(FindJNI) 
        # note: should use JAVA_JVM_LIB_PATH, but it is not set properly 
        set(JAVAVM_LIB ${JAVA_INCLUDE_PATH}/../lib/jvm.lib) 

        set(JAVA_INCLUDE_PATHS ${JAVA_INCLUDE_PATH} ${JAVA_INCLUDE_PATH2}) 
        # message(STATUS "JAVA_INCLUDE_PATHS: " ${JAVA_INCLUDE_PATHS}) 
        # message(STATUS "JAVAVM_LIB: " ${JAVAVM_LIB}) 
      endif(BUILD_JAVA_NATIVE_INTERFACE) 

      set(PM_NEEDED_LIBS winmm.lib)
    endif(WIN32) 
  endif(UNIX)
endif(BUILD_PMDEFAULTS)

# define the jni library  
include_directories(${JAVA_INCLUDE_PATHS})  

set(JNISRC ${PROJECT_SOURCE_DIR}/pm_java/pmjni/pmjni.c)  

# THIS DID NOT WORK - REMOVE WHEN pmjni on Windows works again:
#
# note: PM_LIBSRC is all source for PortMidi, which becomes part of pmjni.
# This means there is no dependency on a separate PortMidi library, and
# pmjni can access internal PortMidi implementation variables and not
# just the "public" API.
message(STATUS "PM_LIBSRC is ${PM_LIBSRC}")
add_library(pmjni SHARED ${JNISRC} ${PM_LIBSRC})
target_link_libraries(pmjni ${PM_NEEDED_LIBS})


set_target_properties(pmjni PROPERTIES  
                            LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})  
set_target_properties(pmjni PROPERTIES EXECUTABLE_EXTENSION "jnilib"
                             MACOSX_RPATH ON)  
set(PMJNI_LIBRARY pmjni)  

