# Elf Crumble [102]

In this challenge we are given a binary `broken` and some other data files:
```
fragment_1.dat
fragment_2.dat
fragment_3.dat
fragment_4.dat
fragment_5.dat
fragment_6.dat
fragment_7.dat
fragment_8.dat
```

When we run `file broken` we see that it is an ELF executable, but when we run it, it just crashes as the challenge descriptions suggests it will. Looking at the disassembly with `objdump -d ./broken -M intel` (intel syntax because we aren't savages) we see that there are 5 functions in the binary (f1, f2, f3, recover_flag, and main) which have been overwritten with 0x58 which is the pop eax instruction on x86. We can also disassemble the fragments with `objdump -b binary -m i386 -M intel -D fragment_1.dat` and see that they contain pieces of the machine code that we need to copy into the `broken` file.

The first thing we notice is that only 4 of the fragments contain `ret` instructions (`objdump -b binary -m i386 -M intel -D fragment_1.dat | grep ret`) and that fragment 1 actually contains 2 `ret` instructions. This means that fragment 1 must contain the contents of an entire function. We look through the binary and calculate the size of each of the 5 missing functions by looking at the offsets in the disassembly (presumably we could also use objdump or some other utility to get this information from the ELF header as well). We find that the functions have the following sizes and offsets:
```
f1: 316
  0x5ad - 0x6e8

f2: 69
  0x6e9 - 0x72d

f3: 116
  0x72e - 0x7a1

recover_flag: 58
  0x7a2 - 0x7db

main: 248
  0x7dc - 0x8d3
```

Now we calculate the number of bytes inbetween the two `ret` instructions in fragment 1 which turns out to be 69, so we know that it must be `f2`. We also can look at the sizes of the fragments to find that fragment 7 (283 bytes) can only fit in `f1` and therefore fragment 3 (175 bytes) must be part of `main`. Fragment 4 is the only fragment that has a `ret` instruction at the end, so it has to be the end of `main`. We continue to piece these chunks together by size and location of `ret` instructions until we get it down to 2 possible orderings. The order of the fragments in the original binary must be either `7, 8, 1, 5, 6 , 2 , 3 , 4` or  `8, 7, 1, 5, 6 , 2 , 3 , 4`. When we look at the disassembly of fragments 7 and 8, we notice that fragment 8 starts with:
```
0:   55                    push   ebp
1:   89 e5                 mov    ebp,esp
3:   53                    push   ebx
4:   83 ec 10              sub    esp,0x10
```
Which looks a lot like the start of a function where it is setting up the stack frame, where as fragment 7 starts off with:
```
0:   0f b6 12                movzx  edx,BYTE PTR [edx]
3:   88 55 fb                mov    BYTE PTR [ebp-0x5],dl
6:   8b 90 38 00 00 00       mov    edx,DWORD PTR [eax+0x38]
c:   89 d1                   mov    ecx,edx
```
Which seems a bit strange to have at the start of a function. So we conclude that the order is most likely `8, 7, 1, 5, 6 , 2 , 3 , 4`.
Now we have to recombine the fragments into the `broken` executable. We can do this with everyone's favorite binary manipulation tool `dd`. We need to start by finding the offset (in decimal) of the start of `f1` (which is 0x5ad or 1453). Now we can use `dd if=fragment_8.dat of=broken bs=1 seek=1453 count=30 conv=notrunc` and then rinse and repeat with the rest of the fragments.

When we run the binary, it prints the flag:
`welcOOOme`
