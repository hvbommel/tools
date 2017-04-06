[PATCH] Do arm_abi detection for ELFOSABI_GNU binaries

•From: Kyle Huey <me at kylehuey dot com>
•To: gdb-patches at sourceware dot org
•Date: Fri, 27 Mar 2015 13:55:16 -0700
•Subject: [PATCH] Do arm_abi detection for ELFOSABI_GNU binaries
•Authentication-results: sourceware.org; auth=none
  
  On ARM systems, gdb must determine which style of breakpoint to use
(see the comments at the beginning of gdb/arm-linux-tdep.c).  In
arm_gdbarch_init we only attempt to extract the eabi version from the
ELF binary if it is a ELFOSABI_NONE binary.  If the binary is
ELFOSABI_GNU instead, we end up defaulting to the old style OABI
syscall breakpoint instruction.  On a Linux kernel built without
CONFIG_OABI_COMPAT, this triggers a SIGILL in ld when attempting to
execute any ELFOSABI_GNU program.  (e.g.
https://github.com/raspberrypi/linux/issues/766)

gdb/ChangeLog:
    * gdb/arm-tdep.c (arm_gdbarch_init): Perform arm_abi detection on
ELFOSABI_GNU binaries.
---
 gdb/arm-tdep.c | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

diff --git a/gdb/arm-tdep.c b/gdb/arm-tdep.c
index 630a207..830739e 100644
--- a/gdb/arm-tdep.c
+++ b/gdb/arm-tdep.c
@@ -9948,17 +9948,17 @@ arm_gdbarch_init (struct gdbarch_info info,
struct gdbarch_list *arches)

       if (ei_osabi == ELFOSABI_ARM)
         {
           /* GNU tools used to use this value, but do not for EABI
          objects.  There's nowhere to tag an EABI version
          anyway, so assume APCS.  */
           arm_abi = ARM_ABI_APCS;
         }
-      else if (ei_osabi == ELFOSABI_NONE)
+      else if (ei_osabi == ELFOSABI_NONE || ei_osabi == ELFOSABI_GNU)
         {
           int eabi_ver = EF_ARM_EABI_VERSION (e_flags);
           int attr_arch, attr_profile;

           switch (eabi_ver)
         {
         case EF_ARM_EABI_UNKNOWN:
           /* Assume GNU tools.  */

