project(mDNSReponder)
cmake_minimum_required(VERSION 3.5)

if(NOT APPLE)
    add_library(mDNSResponder_interface INTERFACE)
    add_library(mDNSResponder::interface ALIAS mDNSResponder_interface)

    set(MDNS_SOURCES mDNSShared/dnssd_clientshim.c mDNSShared/dnssd_clientlib.c mDNSShared/mDNSDebug.c mDNSShared/GenLinkedList.c
                     mDNSCore/mDNS.c mDNSCore/DNSDigest.c mDNSCore/uDNS.c mDNSCore/DNSCommon.c)

    set(MDNS_HEADERS mDNSCore/mDNSEmbeddedAPI.h mDNSCore/mDNSDebug.h mDNSShared/dns_sd.h mDNSShared/mDNSFeatures.h mDNSShared/utilities/nullability.h)

    target_include_directories(mDNSResponder_interface
            INTERFACE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mDNSCore>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mDNSShared>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mDNSShared/utilities>
            $<INSTALL_INTERFACE:include/mDNSResponder>)

    if(UNIX)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE -DHAVE_IPV6 -DNOT_HAVE_SA_LEN -DUSES_NETLINK -DHAVE_LINUX -DTARGET_OS_LINUX")

        list(APPEND MDNS_HEADERS mDNSCore/mDNSDebug.h mDNSPosix/mDNSPosix.h)
        list(APPEND MDNS_SOURCES mDNSPosix/mDNSPosix.c mDNSPosix/mDNSUNP.c mDNSShared/PlatformCommon.c)
        target_include_directories(mDNSResponder_interface
                INTERFACE
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mDNSPosix>)
    elseif(WIN32)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DWIN32_LEAN_AND_MEAN -DNOT_HAVE_SA_LEN -DTARGET_OS_WIN32")

        list(APPEND MDNS_HEADERS mDNSShared/CommonServices.h mDNSCore/uDNS.h mDNSCore/DNSCommon.h mDNSWindows/Poll.h mDNSWindows/mDNSWin32.h mDNSWindows/VisualStudioSupport.h)
        list(APPEND MDNS_SOURCES mDNSWindows/mDNSWin32.c mDNSWindows/Secret.c mDNSWindows/Poll.c mDNSWindows/VisualStudioSupport.cpp)

        target_include_directories(mDNSResponder_interface
                INTERFACE
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mDNSWindows>
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mDNSWindows/SystemService>)
    endif()

    add_library(mDNSResponder STATIC ${MDNS_SOURCES})
    target_link_libraries(mDNSResponder PUBLIC mDNSResponder::interface)

    if(WIN32)
        target_link_libraries(mDNSResponder PRIVATE ws2_32 iphlpapi netapi32 Iphlpapi)
    endif()

    install(TARGETS mDNSResponder
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib)
    install(FILES ${MDNS_HEADERS} DESTINATION include/mDNSResponder)
endif()
