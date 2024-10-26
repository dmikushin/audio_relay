cmake_minimum_required(VERSION 3.10)

# Set the project name
project(audio_relay LANGUAGES NONE)

# Find Docker
find_program(DOCKER_EXECUTABLE NAMES docker)

if(NOT DOCKER_EXECUTABLE)
    message(FATAL_ERROR "Docker not found")
endif()

# Build the Docker container used for all targets below
add_custom_target(
    ${PROJECT_NAME}
    COMMAND ${DOCKER_EXECUTABLE} build -t ${PROJECT_NAME} -f ${CMAKE_CURRENT_SOURCE_DIR}/docker/Dockerfile .
)

# Build & install the server
file(GLOB_RECURSE SERVER_SRCS "${CMAKE_CURRENT_SOURCE_DIR}/server/*")
add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/listen ${CMAKE_CURRENT_BINARY_DIR}/server_pulse
    COMMAND ${DOCKER_EXECUTABLE} build -t ${PROJECT_NAME} -f ${CMAKE_CURRENT_SOURCE_DIR}/docker/Dockerfile .
    COMMAND ${DOCKER_EXECUTABLE} run --rm -v "${CMAKE_CURRENT_SOURCE_DIR}/server:/server" -v "${CMAKE_CURRENT_BINARY_DIR}/target:/server/target" -w "/server" -v "${CMAKE_CURRENT_BINARY_DIR}/cargo:/root/.cargo" ${PROJECT_NAME} cargo build --release
    COMMAND ${DOCKER_EXECUTABLE} run --rm -v "${CMAKE_CURRENT_SOURCE_DIR}/server:/server" -v "${CMAKE_CURRENT_BINARY_DIR}/target:/server/target" -w "/server" -v "${CMAKE_CURRENT_BINARY_DIR}:/usr/local/${PROJECT_NAME}/bin" ${PROJECT_NAME} cargo install --path . --root /usr/local/${PROJECT_NAME} --force
    DEPENDS ${SERVER_SRCS} docker/Dockerfile
)
add_custom_target(
    ${PROJECT_NAME}_server ALL
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/listen ${CMAKE_CURRENT_BINARY_DIR}/server_pulse
)
set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/listen ${CMAKE_CURRENT_BINARY_DIR}/server_pulse PROPERTIES
    GENERATED TRUE
)

# Build the Android native library in Rust
add_custom_target(
    ${PROJECT_NAME}_android_native ALL
    COMMAND ${DOCKER_EXECUTABLE} run --rm -v "${CMAKE_CURRENT_SOURCE_DIR}/server:/server" -v "${CMAKE_CURRENT_BINARY_DIR}/target:/server/target" -w "/server" -v /var/run/docker.sock:/var/run/docker.sock ${PROJECT_NAME} cross build --release --target=aarch64-linux-android
    DEPENDS ${PROJECT_NAME}
)

# Create the gradlew script for Android project
add_custom_target(
    ${PROJECT_NAME}_android_gradlew ALL
    COMMAND ${DOCKER_EXECUTABLE} run --rm -u gradle -v "${CMAKE_CURRENT_SOURCE_DIR}:/app" -w "/app" gradle gradle wrapper
)

# Build the Android APK
add_custom_target(
    ${PROJECT_NAME}_android_apk ALL
    COMMAND ${DOCKER_EXECUTABLE} run --rm -v "${CMAKE_CURRENT_BINARY_DIR}:/app/build" -v "${CMAKE_CURRENT_SOURCE_DIR}:/app" -w "/app" -v "${CMAKE_CURRENT_BINARY_DIR}/target:/target" mobiledevops/android-sdk-image:34.0.1 ./gradlew build
    DEPENDS ${PROJECT_NAME}_android_native ${PROJECT_NAME}_android_gradlew
)
