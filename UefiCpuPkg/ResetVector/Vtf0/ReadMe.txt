
=== HOW TO USE VTF0 ===
Add this line to your DSC [Components.IA32] or [Components.X64] section:
  UefiCpuPkg/ResetVector/Vtf0/ResetVector.inf

Add this line to your FDF FV section:
  INF  RuleOverride=RESET_VECTOR UefiCpuPkg/ResetVector/Vtf0/ResetVector.inf

In your FDF FFS file rules sections add:
  [Rule.Common.SEC.RESET_VECTOR]
    FILE RAW = $(NAMED_GUID) {
      RAW BIN   |.bin
    }

=== VTF0 Boot Flow ===

1. Transition to IA32 flat mode
2. Locate BFV (Boot Firmware Volume) by checking every 4kb boundary
3. Locate SEC image
4. X64 VTF0 transitions to X64 mode
5. Call SEC image entry point

== VTF0 SEC input parameters ==

All inputs to SEC image are register based:
EAX/RAX - Initial value of the EAX register (BIST: Built-in Self Test)
DI      - 'BP': boot-strap processor, or 'AP': application processor
EBP/RBP - Pointer to the start of the Boot Firmware Volume
