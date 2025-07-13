
## 1. Code Formatting

**clang-format Configuration:**

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
TabWidth: 4
UseTab: Never
PointerAlignment: Left
DerivePointerAlignment: false
SpacesInConditionalStatement: false
ColumnLimit: 220
BreakBeforeBraces: Custom
BraceWrapping:
  AfterFunction: false
  AfterClass: false
  AfterStruct: false
  AfterUnion: false
  AfterControlStatement: Multiline
  BeforeElse: false
```

## 2. Naming Conventions

**Functions:**

```c
// Pattern: krs_module_category_functionality
Frame_t krs_frame_create(const uint8_t* buffer, uint16_t received_bytes, uint8_t* stack_data_out, uint16_t stack_data_out_size);
Address_t krs_network_address_ipv4_create(const char* ip);
krs_error_t krs_frame_destroy(Frame_t** frame);
```

**Types:**

```c
// Pattern: TypeName_t
typedef struct Frame Frame_t;
typedef struct Address Address_t;
typedef uint8_t Channel_t;
typedef uint16_t Port_t;
typedef enum krs_error krs_error_t;
```

**Enums:**

```c
// Pattern: EnumName_t mit SCREAMING_SNAKE_CASE values
typedef enum FrameType FrameType_t;
enum FrameType {
    MESSAGE_ACK = 0,
    BASIC_MESSAGE = 1,
    CONNECTION = 10
};
```

**Constants:**

```c
// Pattern: MODULE_CONSTANT_NAME
#define KRONOS_FRAME_HEADER_LENGTH 14
#define KRONOS_BUFFER_SIZE 1024
// Pattern for max values: MAX_NAME_TYPE
#define MAX_CHANNEL_NUMBER 255
#define MAX_IPV4_LENGTH 16
```

## 3. Project Structure

```
kronos/
├── include/          # Public API Headers
│   ├── kronos.h
│   ├── kronos_network.h
│   ├── kronos_server.h
│   ├── kronos_client.h
│   ├── kronos_math.h
│   └── kronos_error.h
├── internal/         # Private Implementation Headers
│   ├── kronos_internal.h
│   ├── frame_header.h
│   ├── frame_body.h
│   ├── network_internal.h
│   └── server_internal.h
├── src/             # Implementation Files
│   ├── frame/
│   ├── math/
│   ├── connection/
│   │   ├── network/
│   │   └── server/
│   └── error/
└── CMakeLists.txt
```

**CMake Include Configuration:**

```cmake
target_include_directories(kronos PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_include_directories(kronos PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/internal
)
```

## 4. Error Handling System

**Error Code Definition with HTTP-Style Value Ranges:**

```c
// kronos_error.h
typedef enum {
    // Success
    KRS_SUCCESS = 0,
    
    // General Errors (1-99)
    KRS_ERR_NULL_POINTER = 1,
    KRS_ERR_INVALID_PARAMETER = 2,
    KRS_ERR_MEMORY_ALLOCATION = 3,
    KRS_ERR_BUFFER_TOO_SMALL = 4,
    KRS_ERR_NOT_INITIALIZED = 5,
    KRS_ERR_ALREADY_INITIALIZED = 6,
    KRS_ERR_NOT_NULL_TERMINATED = 7,
    KRS_ERR_TOO_LONG = 8,
    
    // Frame Errors (100-199)
    KRS_ERR_FRAME_INVALID_HEADER = 100,
    KRS_ERR_FRAME_INVALID_PROTOCOL = 101,
    KRS_ERR_FRAME_CORRUPT_DATA = 102,
    KRS_ERR_FRAME_UNSUPPORTED_VERSION = 103,
    KRS_ERR_FRAME_INVALID_TYPE = 104,
    KRS_ERR_FRAME_BODY_TOO_LARGE = 105,
    KRS_ERR_FRAME_ALREADY_FREED = 106,
    
    // Network Errors (200-299)
    KRS_ERR_NETWORK_INVALID_IP = 200,
    KRS_ERR_NETWORK_INVALID_PORT = 201,
    KRS_ERR_NETWORK_CONNECTION_FAILED = 202,
    KRS_ERR_NETWORK_SOCKET_ERROR = 203,
    KRS_ERR_NETWORK_TIMEOUT = 204,
    KRS_ERR_NETWORK_ADDRESS_IN_USE = 205,
    KRS_ERR_NETWORK_UNREACHABLE = 206,
    
    // Server Errors (300-399)
    KRS_ERR_SERVER_PORT_IN_USE = 300,
    KRS_ERR_SERVER_MAX_CONNECTIONS = 301,
    KRS_ERR_SERVER_NOT_LISTENING = 302,
    KRS_ERR_SERVER_BIND_FAILED = 303,
    KRS_ERR_SERVER_ALREADY_RUNNING = 304,
    
    // Client Errors (400-499)
    KRS_ERR_CLIENT_NOT_CONNECTED = 400,
    KRS_ERR_CLIENT_CONNECTION_LOST = 401,
    KRS_ERR_CLIENT_HANDSHAKE_FAILED = 402,
    KRS_ERR_CLIENT_AUTH_FAILED = 403,
    KRS_ERR_CLIENT_TIMEOUT = 404,
    
    // Math/Utility Errors (500-599)
    KRS_ERR_MATH_OVERFLOW = 500,
    KRS_ERR_MATH_UNDERFLOW = 501,
    KRS_ERR_MATH_INVALID_BITMASK = 502,
    KRS_ERR_MATH_DIVISION_BY_ZERO = 503,
    
    // Platform Errors (600-699)
    KRS_ERR_PLATFORM_WINDOWS_SOCKET = 600,
    KRS_ERR_PLATFORM_LINUX_EPOLL = 650,
    KRS_ERR_PLATFORM_UNSUPPORTED = 699
    
} krs_error_t;
```

**Error Utility Functions:**

```c
// Error category enumeration
typedef enum {
    KRS_ERR_CATEGORY_SUCCESS,
    KRS_ERR_CATEGORY_GENERAL,
    KRS_ERR_CATEGORY_FRAME,
    KRS_ERR_CATEGORY_NETWORK,
    KRS_ERR_CATEGORY_SERVER,
    KRS_ERR_CATEGORY_CLIENT,
    KRS_ERR_CATEGORY_MATH,
    KRS_ERR_CATEGORY_PLATFORM
} krs_error_category_t;

// Error utility functions
const char* krs_error_get_message(krs_error_t error);
krs_error_category_t krs_error_get_category(krs_error_t error);
bool krs_error_is_fatal(krs_error_t error);
```

## 5. Function Variants and Return Patterns

**Dual API Pattern: Simple + Safe Variants**

**Simple Functions (Fast, {0} on error):**

```c
// Returns invalid struct on error - für 90% der use cases
Frame_t krs_frame_create(const uint8_t* buffer, uint16_t received_bytes, 
                        uint8_t* stack_data_out, uint16_t stack_data_out_size);
Address_t krs_network_address_ipv4_create(const char* ip);
```

**Safe Functions (_s suffix, explicit error handling):**

```c
// Result Pattern for functions returning values
typedef struct {
    Frame_t frame;
    krs_error_t error;
    bool valid;
} FrameResult_t;

FrameResult_t krs_frame_create_s(const uint8_t* buffer, uint16_t received_bytes,
                                uint8_t* stack_data_out, uint16_t stack_data_out_size);

typedef struct {
    Address_t address;
    krs_error_t error;
    bool valid;
} AddressResult_t;

AddressResult_t krs_network_address_ipv4_create_s(const char* ip);

// Error Code Return for void functions
krs_error_t krs_frame_destroy_s(Frame_t** frame);
krs_error_t krs_server_shutdown_s(ServerPortManager_t* manager);
```

**String Function Variants:**

```c
// Standard: Null-terminierte Strings (for String Literals("String Literal"))
Address_t krs_network_address_create(const char* ip);
AddressResult_t krs_network_address_create_s(const char* ip);

// Buffer: String of unknown length in unknown length buffer (for Network/File Input)
Address_t krs_network_address_create_buf(const char* ip, size_t buffer_size);
AddressResult_t krs_network_address_create_buf_s(const char* ip, size_t buffer_size);

// Length: String-Length known (for better performance)
Address_t krs_network_address_create_len(const char* ip, size_t ip_len);
AddressResult_t krs_network_address_create_len_s(const char* ip, size_t ip_len);
```

**Memory Management Patterns:**

```c
// Stack-Allocation
Frame_t krs_frame_create(const uint8_t* buffer, uint16_t received_bytes, 
                        uint8_t* stack_data_out, uint16_t stack_data_out_size);
FrameResult_t krs_frame_create_s(const uint8_t* buffer, uint16_t received_bytes,
                                uint8_t* stack_data_out, uint16_t stack_data_out_size);

// Heap-Allocation
Frame_t* krs_frame_create_heap(const uint8_t* buffer, uint16_t received_bytes);
typedef struct {
    Frame_t* frame;
    krs_error_t error;
} FrameHeapResult_t;

FrameHeapResult_t krs_frame_create_heap_s(const uint8_t* buffer, uint16_t received_bytes);

// In-place Initialization (Pre-allocated struct)
void krs_frame_init(const uint8_t* buffer, uint16_t received_bytes, 
                   Frame_t* out, uint16_t out_data_size);
krs_error_t krs_frame_init_s(const uint8_t* buffer, uint16_t received_bytes,
                            Frame_t* out, uint16_t out_data_size);
```

## 6. Parameter Conventions

**Const Correctness:**

```c
// Input-Parameter: const
Address_t krs_network_address_create(const char* ip);
uint16_t krs_frame_get_content(const Frame_t* frame, uint8_t* out, uint16_t out_size);

// Output-Parameter: mutable
krs_error_t krs_frame_init_s(const uint8_t* buffer, uint16_t received_bytes, 
                            Frame_t* out, uint16_t out_data_size);
```

**Size Types:**

```c
// Use size_t for lengths and sizes
// Use length when referring to length of actual information/data
// Use size when referring to the reserved capacity of buffers, arrays and pointers
Address_t krs_network_address_create_buf(const char* ip, size_t buffer_size);

// use specific types for network/protocol constraints
typedef uint8_t Channel_t;    // Hardware-Limit: 0-255
typedef uint16_t Port_t;      // Network-Standard: 0-65535
typedef uint64_t PacketId_t;  // Protocol-Field: 64-bit
```

**Parameter Order Convention:**

```c
// Pattern: Source struct with size, Destination, Destination Size, Destination capacity filled (like memcpy)
krs_error_t krs_frame_get_content_s(const Frame_t* frame, uint8_t* out, 
                                   size_t out_size, size_t* bytes_written);

// Pattern: Source struct with size, Destination struct with size, Size (like memcpy)
krs_error_t krs_frame_copy_content(const Frame_t* src, Frame_t* dest, size_t max_size);

// Pattern: Source, Source Size, Destination, Destination Size, Destination capacity filled or length of data to copy (size_t* data_copied/size_t data_length_to_copy)
// When using size_t data_length_to_copy you can also add a size_t offset parameter
krs_error_t krs_buffer_copy(const uint8_t* source, size_t source_size uint8_t* out, size_t out_size, size_t data_length_to_copy)
```

## 7. Memory Safety

**Safe String Operations:**

```c
AddressResult_t krs_network_address_create_buf_s(const char* ip, size_t buffer_size) {
    AddressResult_t result = {0};
    
    if (ip == NULL) {
        result.error = KRS_ERR_NULL_POINTER;
        result.valid = false;
        return result;
    }
    
    // Sichere Längenprüfung mit strnlen
    size_t ip_len = strnlen(ip, buffer_size);
    if (ip_len == buffer_size) {
        result.error = KRS_ERR_NOT_NULL_TERMINATED;
        result.valid = false;
        return result;
    }
    if (ip_len >= MAX_IPV4_LENGTH) {
        result.error = KRS_ERR_TOO_LONG;
        result.valid = false;
        return result;
    }
    
    // Success path
    result.address = krs_network_address_create_len(ip, ip_len);
    result.error = KRS_SUCCESS;
    result.valid = true;
    return result;
}
```

**Defensive Programming:**

```c
krs_error_t krs_frame_destroy_s(Frame_t** frame) {
    if (frame == NULL) return KRS_ERR_NULL_POINTER;
    if (*frame == NULL) return KRS_SUCCESS; // Already freed is OK
    
    if ((*frame)->body != NULL) {
        free((*frame)->body);
        (*frame)->body = NULL;  // Avoid dangling pointer
    }
    
    free(*frame);
    *frame = NULL;  // Avoid dangling pointer
    return KRS_SUCCESS;
}
```

**Buffer Bounds Checking:**

```c
krs_error_t krs_frame_get_content_s(const Frame_t* frame, uint8_t* out, 
                                   size_t out_size, size_t* bytes_written) {
    if (frame == NULL || out == NULL) return KRS_ERR_NULL_POINTER;
    if (bytes_written != NULL) *bytes_written = 0;
    
    if (frame->body_length > out_size) return KRS_ERR_BUFFER_TOO_SMALL;
    
    memcpy(out, frame->body, frame->body_length);
    if (bytes_written != NULL) *bytes_written = frame->body_length;
    
    return KRS_SUCCESS;
}
```

## 8. Header Organization

**Public Headers (include/):**

```c
#ifndef KRONOS_NETWORK_H
#define KRONOS_NETWORK_H

#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>
#include "kronos_error.h"

// Forward declarations - no includes of internal headers
typedef struct Address Address_t;
typedef struct PortAddress PortAddress_t;

// Result types
typedef struct {
    Address_t address;
    krs_error_t error;
    bool valid;
} AddressResult_t;

// Function declarations
Address_t krs_network_address_ipv4_create(const char* ip);
AddressResult_t krs_network_address_ipv4_create_s(const char* ip);

#endif // KRONOS_NETWORK_H
```

**Internal Headers (internal/):**

```c
#ifndef NETWORK_INTERNAL_H
#define NETWORK_INTERNAL_H

#include "kronos_network.h"  // Include corresponding public header
#include <winsock2.h>        // Platform-specific includes

// Actual struct definitions
struct Address {
    struct in6_addr addr;
    krs_error_t last_error;
    bool is_valid;
};

// Internal function declarations
krs_error_t krs_network_internal_validate_ipv4(const char* ip, size_t len);

#endif // NETWORK_INTERNAL_H
```

**Include Guidelines:**

```c
// In source files: Include public header first, then internal
#include "kronos_network.h"
#include "network_internal.h"
#include <standard_library_headers.h>
#include <platform_specific_headers.h>
```

## 9. Documentation Standards

**Public API Functions:**

```c
/**
 * @brief Creates an IPv4 address from a string with explicit error handling.
 * 
 * @param ip        Null-terminated IPv4 string (e.g., "192.168.1.1")
 * @param buffer_size Maximum size of the ip buffer including null terminator
 * @return AddressResult_t structure containing address or error information
 * 
 * @note The ip parameter must be null-terminated within buffer_size bytes.
 * @warning This function validates buffer bounds but does not validate IP format.
 * 
 * @retval KRS_SUCCESS Successfully created address
 * @retval KRS_ERR_NULL_POINTER ip parameter is NULL
 * @retval KRS_ERR_NOT_NULL_TERMINATED String not terminated within buffer_size
 * @retval KRS_ERR_TOO_LONG IP string exceeds maximum length
 * @retval KRS_ERR_NETWORK_INVALID_IP Invalid IP address format
 */
AddressResult_t krs_network_address_ipv4_create_buf_s(const char* ip, size_t buffer_size);
```

**Simple Function Documentation:**

```c
/**
 * @brief Creates an IPv4 address from a string (simple version).
 * 
 * @param ip Null-terminated IPv4 string (e.g., "192.168.1.1")
 * @return Address_t structure, or invalid address on error
 * 
 * @note Use krs_network_address_ipv4_create_s() for explicit error handling.
 * @note Returns invalid address (all zeros) on any error condition.
 */
Address_t krs_network_address_ipv4_create(const char* ip);
```

## 10. Build System Best Practices

**CMake Configuration:**

```cmake
# Minimum version and project setup
cmake_minimum_required(VERSION 3.15)
project(kronos VERSION 1.0.0 LANGUAGES C)

# Standards
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Compiler warnings
if(MSVC)
    target_compile_options(kronos PRIVATE /W4 /WX)
else()
    target_compile_options(kronos PRIVATE -Wall -Wextra -Wpedantic -Werror)
endif()

# Static analysis in debug builds
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    find_program(CLANG_TIDY_EXE NAMES "clang-tidy")
    if(CLANG_TIDY_EXE)
        set_target_properties(kronos PROPERTIES C_CLANG_TIDY "${CLANG_TIDY_EXE}")
    endif()
endif()

# Error handling source files
target_sources(kronos PRIVATE
    src/error/error_messages.c
    src/error/error_categories.c
)
```

## 11. Performance Guidelines

**Buffer Reuse Pattern:**

```c
// Context-based operations for Performance
typedef struct {
    uint8_t frame_buffer[KRONOS_BUFFER_SIZE];
    uint8_t work_buffer[256];
    char string_buffer[128];
} krs_context_t;

krs_error_t krs_context_init(krs_context_t* ctx);
FrameResult_t krs_frame_create_ctx_s(krs_context_t* ctx, const uint8_t* network_data, uint16_t data_len);
```

**Minimize Allocations:**

```c
// Prefer stack allocation for small, temporary data
char ip_buffer[INET6_ADDRSTRLEN];
AddressResult_t result = krs_network_address_create_buf_s(ip_buffer, sizeof(ip_buffer));

// Use heap allocation only for large or persistent data
FrameHeapResult_t heap_result = krs_frame_create_heap_s(large_network_packet, packet_size);
```

## 12. Platform Compatibility

**Conditional Compilation:**

```c
// Platform-specific types
#ifdef _WIN32
    #include <winsock2.h>
    typedef SOCKET UDPSocketRef_t;
    #define KRS_SOCKET_INVALID INVALID_SOCKET
#else
    #include <sys/socket.h>
    typedef int UDPSocketRef_t;
    #define KRS_SOCKET_INVALID -1
#endif

// Platform-specific error codes
#ifdef _WIN32
    #define KRS_GET_LAST_SOCKET_ERROR() WSAGetLastError()
#else
    #define KRS_GET_LAST_SOCKET_ERROR() errno
#endif
```

**CMake Platform Handling:**

```cmake
# Platform-specific linking
if(WIN32)
    target_link_libraries(kronos PRIVATE ws2_32)
    target_compile_definitions(kronos PRIVATE KRS_PLATFORM_WINDOWS)
else()
    target_link_libraries(kronos PRIVATE pthread)
    target_compile_definitions(kronos PRIVATE KRS_PLATFORM_UNIX)
endif()
```