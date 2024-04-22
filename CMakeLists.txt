project(mDNSReponder)
cmake_minimum_required(VERSION 3.5)

if(NOT APPLE)
    set(MDNS_SOURCES mDNSShared/dnssd_clientshim.c mDNSShared/dnssd_clientlib.c mDNSShared/mDNSDebug.c mDNSShared/GenLinkedList.c
                     mDNSCore/mDNS.c mDNSCore/DNSDigest.c mDNSCore/uDNS.c mDNSCore/DNSCommon.c)

    set(MDNS_HEADERS mDNSCore/mDNSEmbeddedAPI.h mDNSCore/mDNSDebug.h mDNSShared/mDNSFeatures.h mDNSShared/utilities/nullability.h)

    if(UNIX)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE -DHAVE_IPV6 -DNOT_HAVE_SA_LEN -DUSES_NETLINK -DHAVE_LINUX -DTARGET_OS_LINUX")

        list(APPEND MDNS_HEADERS mDNSCore/mDNSDebug.h mDNSPosix/mDNSPosix.h)
        list(APPEND MDNS_SOURCES mDNSPosix/mDNSPosix.c mDNSPosix/mDNSUNP.c mDNSShared/PlatformCommon.c)
    elseif(WIN32)
        include_directories(mDNSWindows/SystemService ${CO_SAFEC_INCLUDE_DIR})

        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DWIN32_LEAN_AND_MEAN -DNOT_HAVE_SA_LEN -DTARGET_OS_WIN32")
        if(NOT MSVC)
            string(APPEND CMAKE_C_FLAGS " -DHAVE_SAFEC")
        endif()

        list(APPEND MDNS_HEADERS mDNSShared/CommonServices.h mDNSCore/uDNS.h mDNSCore/DNSCommon.h mDNSWindows/Poll.h mDNSWindows/mDNSWin32.h mDNSWindows/VisualStudioSupport.h)
        list(APPEND MDNS_SOURCES mDNSWindows/mDNSWin32.c mDNSWindows/Secret.c mDNSWindows/Poll.c mDNSWindows/VisualStudioSupport.cpp)
        include_directories(mDNSWindows)
    endif()

    include_directories(mDNSCore mDNSShared mDNSShared/utilities)
    add_library(mDNSResponder ${MDNS_SOURCES})
    if(WIN32)
        target_link_libraries(mDNSResponder ws2_32 iphlpapi netapi32 Iphlpapi ${CO_SAFEC_LIB})
    endif()

    install(TARGETS mDNSResponder
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib)
    install(FILES ${MDNS_HEADERS} DESTINATION include/mDNSResponder)
    # Simplify typical include pattern here (include instead of include/mDNSResponder), because
    # on APPLE we use system header
    install(FILES mDNSShared/dns_sd.h DESTINATION include)
endif()