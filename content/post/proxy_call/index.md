---
title: function call page memory auto mapping
description: Managing Function Calls Across Paged Memory on the Spectrum Next.
slug: proxy_call
date: 2025-10-30 00:00:00+0000
image: cover.jpg
categories:
    - general
tags:
    - development
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

During the resolution of my core engine growing beyond 8 KB, I analyzed the impact of splitting the engine into multiple functional blocks — for example, physics, render, and input. While exploring that approach, I also considered allowing the engine core to page itself in and out, dynamically loading the needed memory bank before calling a specific function. Although that design was later dropped, I decided to keep the prototype paging routine for a similar purpose.

The goal is to call functions that may reside in different memory pages, automatically page in the required 8 KB block, execute the function, and then restore the previous mapping. This approach allows managing up to eight 8 KB pages (64 KB total), each mapped into bank #0 as needed.

## Address Structure ##

As mentioned in a previous article, the Z80 address bus is 16-bit, which allows addressing 64 KB at once. To identify both the memory page and the function address, we can split the 16-bit address this way:

| Bits  | Meaning          | Range    |
| ----- | ---------------- | -------- |
| 15–13 | Page index       | 0 – 7    |
| 12–0  | Function address | 0 – 8191 |

Each 8 KB page contains 8192 bytes (2¹³), so the top 3 bits of a 16-bit value can define which page to load, and the lower 13 bits specify the address within that page.

This means functions must be compiled to execute in bank #0, since only that slot is dynamically paged.

## Function definition example ##

```text
mmu $0000, DEBUG_BANK
    org $0000

@Proxy_Show_Collider equ  (Proxy_Bank<<13)+Debug_Show_Collider

Debug_Show_Collider:
    ld    hl, Collider_Active_Cache
.Loop:
    ld    a, (hl)
    ld    iyl, a
    inc   hl
    ld    a, (hl) 
    inc   hl
    ld    iyh, a
    or    iyl
    ret   z
    push  hl
    call  Debug.DrawCollider
    pop   hl
    jr    .Loop
```

The function itself isn’t important — what matters is the structure. The label @Proxy_Show_Collider follows the defined format: top 3 bits = page index, remaining 13 bits = function address.

## Example Call ##

```text
    call  ProxyCall
    dw    @Proxy_Show_Collider
    call  CleanData
```

The ProxyCall routine interprets the function pointer, pages in the correct page into bank #0, calls the target function, and restores the previous page afterward.

## The ProxyCall Routine ##

```text
;------------------------------------------------------------
; ProxyCall
; Dynamically pages in the target bank and calls its function
;------------------------------------------------------------
;
; Expected:
;   (SP) → proxy call function address (ppp aaaaaaaaaaaaa)
;           p = page 0..7, a = 0..8191
;
ProxyCall:
    pop   hl
    ld    e, (hl)
    inc   hl
    ld    d, (hl)
    inc   hl
    push  hl 
```

We use a call instruction to reach this routine, so the next instruction’s address (the return address) is already on the stack, by popping it into HL, reading the following word into DE, and pushing HL back, we obtain the target function address while preserving the return address.

```text
    ld    hl, (SLOT_0000)
    push  hl                
    push  af
    ex    de, hl
    ld    a, h
    add   a, a
    adc   a, a
    adc   a, a
    adc   a, a
    and   %00000111
    add   PROXY_BASE_BANK
    call  setbank0000_8k
```

Here we store the current page in use by bank #0, preserve register A, extract the page index from the top bits of H, and map the corresponding page.

```text
    ld    a, h
    and   %00011111
    ld    h, a
```

his ensures that only the lower 13 bits of HL remain — the actual function address.

```text
    pop   af
    call  Call_hl
```

After restoring register A, we call Call_hl, which is a simple jp (hl) — effectively acting as a call (hl) pseudo-opcode.

```text
    pop   hl
    jp    setbanks0000_hl
```

Finally, we restore the original bank to slot #0 and return execution to the next instruction (call CleanData in the example).

## Example of Equivalent Inline Bank Switch ##

```text
    ld    a, (SLOT_0000)
    push  af
    ld    a, $00
    call  setbank0000_8k
    ;
    ; routine code
    ;
    pop   af
    call  setbank0000_8k
    ret
```

## Performance Notes ##

| Call Type      | Size     | T-States     |
| -------------- | -------- | ------------ |
| ProxyCall      | 31 bytes | 167 t-states |
| Regular call   | 14 bytes | 85 t-states  |

The proxy adds roughly 82 extra t-states but saves 11 bytes per function, which is a reasonable trade-off for infrequently called routines.

---

## Closing Thoughts ##

This proxy-based approach adds flexibility for managing large engines that exceed 8 KB while keeping the main memory footprint small. Although not ideal for performance-critical routines, it’s a clean and modular way to access code spread across multiple pages — an elegant solution for large Z80 projects like the Spectrum Next.