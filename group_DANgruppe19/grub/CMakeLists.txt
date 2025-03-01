########################################
###
### Create a CMAKE Target for grub-mkrescue
### We will use grub-mkrescue to create a bootable image for our kernel
########################################


# Set target and path for the image
set(GRUB_IMAGE_EXTENSION img)
set(GRUB_TARGET ${OS_KERNEL_NAME}-create-image)
set(GRUB_IMAGE_PATH ${CMAKE_BINARY_DIR}/kernel.${GRUB_IMAGE_EXTENSION})

if(OS_ARCH_TARGET STREQUAL "i386")
    set(GRUB_MULTIBOOT_VERSION multiboot)
elseif(OS_ARCH_TARGET STREQUAL "x86_64")
    set(GRUB_MULTIBOOT_VERSION multiboot2)
endif()


find_program(GRUB_MKRESCUE grub-mkrescue)
if (NOT GRUB_MKRESCUE)
    message (FATAL_ERROR "Unable to find `grub-mkrescue`. Bootable ISO image generation will not be available.")
endif()

if(NOT DEFINED OS_NAME)
    message(FATAL_ERROR "You must set OS_NAME variable")
endif()

if(NOT DEFINED OS_KERNEL_NAME)
    message(FATAL_ERROR "You must set OS_KERNEL_NAME variable")
endif()

# Create file structure for the bootable image
file(REMOVE ${CMAKE_SOURCE_DIR}/iso)
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/iso/bin)
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/iso/boot/grub)

# Configure grub.cfg
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/grub.cfg.in ${CMAKE_BINARY_DIR}/iso/boot/grub/grub.cfg)

# Configure disk.sh
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/disk.sh.in ${CMAKE_BINARY_DIR}/disk.sh)


add_custom_target(${GRUB_TARGET}
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:${OS_KERNEL_NAME}-kernel> ${CMAKE_BINARY_DIR}/iso/boot/
        COMMAND ${GRUB_MKRESCUE} --output=${GRUB_IMAGE_PATH} ${CMAKE_BINARY_DIR}/iso
        COMMAND /bin/sh ${CMAKE_BINARY_DIR}/disk.sh disk.iso
)

########################################
###
### Export variables to parent_Scope
###
########################################
set(GRUB_TARGET ${GRUB_TARGET} PARENT_SCOPE)
set(GRUB_IMAGE_PATH ${GRUB_IMAGE_PATH} PARENT_SCOPE)


