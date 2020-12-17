# AOA

Premier Code : 

double dotprod(double *restrict a, double *restrict b, unsigned long long n)
{
double d = 0.0;
for (unsigned long long i = 0; i < n; i++)
d += a[i] * b[i];
return d;
}

X86-64 gcc 10.2 :  sans flag
dotprod:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-24], rdi
        mov     QWORD PTR [rbp-32], rsi
        mov     QWORD PTR [rbp-40], rdx
        pxor    xmm0, xmm0
        movsd   QWORD PTR [rbp-8], xmm0
        mov     QWORD PTR [rbp-16], 0
        jmp     .L2
.L3:
        mov     rax, QWORD PTR [rbp-16]
        lea     rdx, [0+rax*8]
        mov     rax, QWORD PTR [rbp-24]
        add     rax, rdx
        movsd   xmm1, QWORD PTR [rax]
        mov     rax, QWORD PTR [rbp-16]
        lea     rdx, [0+rax*8]
        mov     rax, QWORD PTR [rbp-32]
        add     rax, rdx
        movsd   xmm0, QWORD PTR [rax]
        mulsd   xmm0, xmm1
        movsd   xmm1, QWORD PTR [rbp-8]
        addsd   xmm0, xmm1
        movsd   QWORD PTR [rbp-8], xmm0
        add     QWORD PTR [rbp-16], 1
.L2:
        mov     rax, QWORD PTR [rbp-16]
        cmp     rax, QWORD PTR [rbp-40]
        jb      .L3
        movsd   xmm0, QWORD PTR [rbp-8]
        movq    rax, xmm0
        movq    xmm0, rax
        pop     rbp
        ret

-O1:
dotprod:
        test    rdx, rdx
        je      .L4
        mov     eax, 0
        pxor    xmm1, xmm1
.L3:
        movsd   xmm0, QWORD PTR [rdi+rax*8]
        mulsd   xmm0, QWORD PTR [rsi+rax*8]
        addsd   xmm1, xmm0
        add     rax, 1
        cmp     rdx, rax
        jne     .L3
.L1:
        movapd  xmm0, xmm1
        ret
.L4:
        pxor    xmm1, xmm1
        jmp     .L1

-O2 :  est plus sensible au scheduling organisation des instructions
dotprod:
        test    rdx, rdx
        je      .L4
        xor     eax, eax
        pxor    xmm1, xmm1
.L3:
        movsd   xmm0, QWORD PTR [rdi+rax*8]
        mulsd   xmm0, QWORD PTR [rsi+rax*8]
        add     rax, 1
        addsd   xmm1, xmm0
        cmp     rdx, rax
        jne     .L3
        movapd  xmm0, xmm1
        ret
.L4:
        pxor    xmm1, xmm1
        movapd  xmm0, xmm1
        ret

-O3 :  
dotprod:
        test    rdx, rdx
        je      .L7
        cmp     rdx, 1
        je      .L8
        mov     rcx, rdx
        xor     eax, eax
        pxor    xmm1, xmm1
        shr     rcx
        sal     rcx, 4
.L4:
        movupd  xmm0, XMMWORD PTR [rdi+rax]
        movupd  xmm3, XMMWORD PTR [rsi+rax]
        add     rax, 16
        mulpd   xmm0, xmm3
        addsd   xmm1, xmm0
        unpckhpd        xmm0, xmm0
        addsd   xmm1, xmm0
        cmp     rcx, rax
        jne     .L4
        mov     rax, rdx
        and     rax, -2
        and     edx, 1
        je      .L1
.L3:
        movsd   xmm0, QWORD PTR [rsi+rax*8]
        mulsd   xmm0, QWORD PTR [rdi+rax*8]
        addsd   xmm1, xmm0
.L1:
        movapd  xmm0, xmm1
        ret
.L7:
        pxor    xmm1, xmm1
        movapd  xmm0, xmm1
        ret
.L8:
        xor     eax, eax
        pxor    xmm1, xmm1
        jmp     .L3

-Ofast : 
dotprod:
        test    rdx, rdx
        je      .L7
        cmp     rdx, 1
        je      .L8
        mov     rcx, rdx
        xor     eax, eax
        pxor    xmm2, xmm2
        shr     rcx
        sal     rcx, 4
.L4:
        movupd  xmm0, XMMWORD PTR [rdi+rax]
        movupd  xmm3, XMMWORD PTR [rsi+rax]
        add     rax, 16
        mulpd   xmm0, xmm3
        addpd   xmm2, xmm0
        cmp     rcx, rax
        jne     .L4
        mov     rax, rdx
        movapd  xmm1, xmm2
        unpckhpd        xmm1, xmm2
        and     rax, -2
        and     edx, 1
        addpd   xmm1, xmm2
        je      .L1
.L3:
        movsd   xmm0, QWORD PTR [rsi+rax*8]
        mulsd   xmm0, QWORD PTR [rdi+rax*8]
        addsd   xmm1, xmm0
.L1:
        movapd  xmm0, xmm1
        ret
.L7:
        pxor    xmm1, xmm1
        movapd  xmm0, xmm1
        ret
.L8:
        xor     eax, eax
        pxor    xmm1, xmm1
        jmp     .L3

kamikaze:  
fma fait la multiplication et l’addition au même temps optimise beaucoup ( scalaire )
dotprod:
        mov     rcx, rdx
        test    rdx, rdx
        je      .L7
        lea     rax, [rdx-1]
        cmp     rax, 2
        jbe     .L8
        mov     r8, rdx
        shr     r8, 2
        sal     r8, 5
        lea     rdx, [r8-32]
        shr     rdx, 5
        inc     rdx
        xor     r9d, r9d
        vxorpd  xmm0, xmm0, xmm0
        and     edx, 7
        je      .L4
        cmp     rdx, 1
        je      .L31
        cmp     rdx, 2
        je      .L32
        cmp     rdx, 3
        je      .L33
        cmp     rdx, 4
        je      .L34
        cmp     rdx, 5
        je      .L35
        cmp     rdx, 6
        jne     .L48
.L36:
        vmovupd ymm3, YMMWORD PTR [rdi+r9]
        vfmadd231pd     ymm0, ymm3, YMMWORD PTR [rsi+r9]
        add     r9, 32
.L35:
        vmovupd ymm4, YMMWORD PTR [rdi+r9]
        vfmadd231pd     ymm0, ymm4, YMMWORD PTR [rsi+r9]
        add     r9, 32
.L34:
        vmovupd ymm5, YMMWORD PTR [rdi+r9]
        vfmadd231pd     ymm0, ymm5, YMMWORD PTR [rsi+r9]
        add     r9, 32
.L33:
        vmovupd ymm6, YMMWORD PTR [rdi+r9]
        vfmadd231pd     ymm0, ymm6, YMMWORD PTR [rsi+r9]
        add     r9, 32
.L32:
        vmovupd ymm1, YMMWORD PTR [rdi+r9]
        vfmadd231pd     ymm0, ymm1, YMMWORD PTR [rsi+r9]
        add     r9, 32
.L31:
        vmovupd ymm7, YMMWORD PTR [rdi+r9]
        vfmadd231pd     ymm0, ymm7, YMMWORD PTR [rsi+r9]
        add     r9, 32
        cmp     r9, r8
        je      .L45
.L4:
        vmovupd ymm8, YMMWORD PTR [rdi+r9]
        vmovupd ymm9, YMMWORD PTR [rdi+32+r9]
        vfmadd231pd     ymm0, ymm8, YMMWORD PTR [rsi+r9]
        vmovupd ymm10, YMMWORD PTR [rdi+64+r9]
        vmovupd ymm11, YMMWORD PTR [rdi+96+r9]
        vmovupd ymm12, YMMWORD PTR [rdi+128+r9]
        vmovupd ymm13, YMMWORD PTR [rdi+160+r9]
        vfmadd231pd     ymm0, ymm9, YMMWORD PTR [rsi+32+r9]
        vmovupd ymm14, YMMWORD PTR [rdi+192+r9]
        vmovupd ymm15, YMMWORD PTR [rdi+224+r9]
        vfmadd231pd     ymm0, ymm10, YMMWORD PTR [rsi+64+r9]
        vfmadd231pd     ymm0, ymm11, YMMWORD PTR [rsi+96+r9]
        vfmadd231pd     ymm0, ymm12, YMMWORD PTR [rsi+128+r9]
        vfmadd231pd     ymm0, ymm13, YMMWORD PTR [rsi+160+r9]
        vfmadd231pd     ymm0, ymm14, YMMWORD PTR [rsi+192+r9]
        vfmadd231pd     ymm0, ymm15, YMMWORD PTR [rsi+224+r9]
        add     r9, 256
        cmp     r9, r8
        jne     .L4
.L45:
        vextractf64x2   xmm2, ymm0, 0x1
        vaddpd  xmm3, xmm2, xmm0
        mov     r10, rcx
        and     r10, -4
        vunpckhpd       xmm0, xmm3, xmm3
        vaddpd  xmm0, xmm0, xmm3
        test    cl, 3
        je      .L49
        vzeroupper
.L3:
        vmovsd  xmm4, QWORD PTR [rdi+r10*8]
        lea     r11, [r10+1]
        vfmadd231sd     xmm0, xmm4, QWORD PTR [rsi+r10*8]
        cmp     rcx, r11
        jbe     .L1
        vmovsd  xmm5, QWORD PTR [rdi+r11*8]
        add     r10, 2
        vfmadd231sd     xmm0, xmm5, QWORD PTR [rsi+r11*8]
        cmp     rcx, r10
        jbe     .L1
        vmovsd  xmm6, QWORD PTR [rsi+r10*8]
        vfmadd231sd     xmm0, xmm6, QWORD PTR [rdi+r10*8]
        ret
.L7:
        vxorpd  xmm0, xmm0, xmm0
.L1:
        ret
.L49:
        vzeroupper
        ret
.L48:
        vmovupd ymm2, YMMWORD PTR [rdi]
        mov     r9d, 32
        vfmadd231pd     ymm0, ymm2, YMMWORD PTR [rsi]
        jmp     .L36
.L8:
        xor     r10d, r10d
        vxorpd  xmm0, xmm0, xmm0
        jmp     .L3


Deuxième Code : 
double dotprod_unroll2(double *restrict a, double *restrict b, unsigned long long n)
{
double d1 = 0.0;
double d2 = 0.0;
for (unsigned long long i = 0; i < n; i += 2)
{
d1 += (a[i] * b[i]);
d2 += (a[i + 1] * b[i + 1]);
}
return (d1 + d2);
}

Sans flag : 
dotprod_unroll2:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-40], rdi
        mov     QWORD PTR [rbp-48], rsi
        mov     QWORD PTR [rbp-56], rdx
        pxor    xmm0, xmm0
        movsd   QWORD PTR [rbp-8], xmm0
        pxor    xmm0, xmm0
        movsd   QWORD PTR [rbp-16], xmm0
        mov     QWORD PTR [rbp-24], 0
        jmp     .L2
.L3:
        mov     rax, QWORD PTR [rbp-24]
        lea     rdx, [0+rax*8]
        mov     rax, QWORD PTR [rbp-40]
        add     rax, rdx
        movsd   xmm1, QWORD PTR [rax]
        mov     rax, QWORD PTR [rbp-24]
        lea     rdx, [0+rax*8]
        mov     rax, QWORD PTR [rbp-48]
        add     rax, rdx
        movsd   xmm0, QWORD PTR [rax]
        mulsd   xmm0, xmm1
        movsd   xmm1, QWORD PTR [rbp-8]
        addsd   xmm0, xmm1
        movsd   QWORD PTR [rbp-8], xmm0
        mov     rax, QWORD PTR [rbp-24]
        add     rax, 1
        lea     rdx, [0+rax*8]
        mov     rax, QWORD PTR [rbp-40]
        add     rax, rdx
        movsd   xmm1, QWORD PTR [rax]
        mov     rax, QWORD PTR [rbp-24]
        add     rax, 1
        lea     rdx, [0+rax*8]
        mov     rax, QWORD PTR [rbp-48]
        add     rax, rdx
        movsd   xmm0, QWORD PTR [rax]
        mulsd   xmm0, xmm1
        movsd   xmm1, QWORD PTR [rbp-16]
        addsd   xmm0, xmm1
        movsd   QWORD PTR [rbp-16], xmm0
        add     QWORD PTR [rbp-24], 2
.L2:
        mov     rax, QWORD PTR [rbp-24]
        cmp     rax, QWORD PTR [rbp-56]
        jb      .L3
        movsd   xmm0, QWORD PTR [rbp-8]
        addsd   xmm0, QWORD PTR [rbp-16]
        movq    rax, xmm0
        movq    xmm0, rax
        pop     rbp
        ret

-O1 : 
dotprod_unroll2:
        test    rdx, rdx
        je      .L4
        mov     eax, 0
        pxor    xmm2, xmm2
        movapd  xmm1, xmm2
.L3:
        movsd   xmm0, QWORD PTR [rdi+rax*8]
        mulsd   xmm0, QWORD PTR [rsi+rax*8]
        addsd   xmm1, xmm0
        movsd   xmm0, QWORD PTR [rdi+8+rax*8]
        mulsd   xmm0, QWORD PTR [rsi+8+rax*8]
        addsd   xmm2, xmm0
        add     rax, 2
        cmp     rdx, rax
        ja      .L3
.L2:
        addsd   xmm1, xmm2
        movapd  xmm0, xmm1
        ret
.L4:
        pxor    xmm2, xmm2
        movapd  xmm1, xmm2
        jmp     .L2

-O2 : 
dotprod_unroll2:
        test    rdx, rdx
        je      .L4
        pxor    xmm2, xmm2
        xor     eax, eax
        movapd  xmm1, xmm2
.L3:
        movsd   xmm0, QWORD PTR [rdi+rax*8]
        mulsd   xmm0, QWORD PTR [rsi+rax*8]
        addsd   xmm1, xmm0
        movsd   xmm0, QWORD PTR [rdi+8+rax*8]
        mulsd   xmm0, QWORD PTR [rsi+8+rax*8]
        add     rax, 2
        addsd   xmm2, xmm0
        cmp     rdx, rax
        ja      .L3
        addsd   xmm1, xmm2
        movapd  xmm0, xmm1
        ret
.L4:
        pxor    xmm0, xmm0
        ret

-O3 :
dotprod_unroll2:
        test    rdx, rdx
        je      .L6
        sub     rdx, 1
        mov     rcx, rdx
        shr     rcx
        add     rcx, 1
        cmp     rdx, 1
        jbe     .L7
        mov     rdx, rcx
        pxor    xmm1, xmm1
        xor     eax, eax
        shr     rdx
        movapd  xmm3, xmm1
        sal     rdx, 5
.L4:
        movupd  xmm2, XMMWORD PTR [rdi+rax]
        movupd  xmm0, XMMWORD PTR [rsi+rax]
        movhpd  xmm2, QWORD PTR [rdi+16+rax]
        movhpd  xmm0, QWORD PTR [rsi+16+rax]
        mulpd   xmm2, xmm0
        movapd  xmm0, xmm2
        unpckhpd        xmm2, xmm2
        addsd   xmm0, xmm3
        movapd  xmm3, xmm2
        movupd  xmm2, XMMWORD PTR [rsi+16+rax]
        movlpd  xmm2, QWORD PTR [rsi+8+rax]
        addsd   xmm3, xmm0
        movupd  xmm0, XMMWORD PTR [rdi+16+rax]
        movlpd  xmm0, QWORD PTR [rdi+8+rax]
        add     rax, 32
        mulpd   xmm0, xmm2
        addsd   xmm1, xmm0
        unpckhpd        xmm0, xmm0
        addsd   xmm1, xmm0
        cmp     rax, rdx
        jne     .L4
        mov     rdx, rcx
        and     rdx, -2
        lea     rax, [rdx+rdx]
        cmp     rdx, rcx
        je      .L5
.L3:
        movsd   xmm0, QWORD PTR [rsi+rax*8]
        mulsd   xmm0, QWORD PTR [rdi+rax*8]
        addsd   xmm3, xmm0
        movsd   xmm0, QWORD PTR [rsi+8+rax*8]
        mulsd   xmm0, QWORD PTR [rdi+8+rax*8]
        addsd   xmm1, xmm0
.L5:
        addsd   xmm1, xmm3
        movapd  xmm0, xmm1
        ret
.L6:
        pxor    xmm0, xmm0
        ret
.L7:
        pxor    xmm1, xmm1
        xor     eax, eax
        movapd  xmm3, xmm1
        jmp     .L3

-Ofast :
dotprod_unroll2:
        test    rdx, rdx
        je      .L4
        sub     rdx, 1
        xor     eax, eax
        pxor    xmm1, xmm1
        shr     rdx
        lea     rcx, [rdx+1]
        xor     edx, edx
.L3:
        movupd  xmm0, XMMWORD PTR [rsi+rax]
        movupd  xmm3, XMMWORD PTR [rdi+rax]
        add     rdx, 1
        add     rax, 16
        mulpd   xmm0, xmm3
        addpd   xmm1, xmm0
        cmp     rdx, rcx
        jb      .L3
        movapd  xmm4, xmm1
        movapd  xmm0, xmm1
        unpckhpd        xmm4, xmm4
        addsd   xmm0, xmm4
        ret
.L4:
        pxor    xmm0, xmm0
        ret

kamikaze:
dotprod_unroll2:
        test    rdx, rdx
        je      .L6
        dec     rdx
        mov     rcx, rdx
        shr     rcx
        inc     rcx
        cmp     rdx, 1
        jbe     .L7
        mov     r8, rcx
        shr     r8
        sal     r8, 5
        lea     rdx, [r8-32]
        shr     rdx, 5
        inc     rdx
        xor     eax, eax
        vxorpd  xmm0, xmm0, xmm0
        and     edx, 7
        je      .L4
        cmp     rdx, 1
        je      .L30
        cmp     rdx, 2
        je      .L31
        cmp     rdx, 3
        je      .L32
        cmp     rdx, 4
        je      .L33
        cmp     rdx, 5
        je      .L34
        cmp     rdx, 6
        jne     .L47
.L35:
        vmovupd ymm4, YMMWORD PTR [rsi+rax]
        vfmadd231pd     ymm0, ymm4, YMMWORD PTR [rdi+rax]
        add     rax, 32
.L34:
        vmovupd ymm7, YMMWORD PTR [rsi+rax]
        vfmadd231pd     ymm0, ymm7, YMMWORD PTR [rdi+rax]
        add     rax, 32
.L33:
        vmovupd ymm6, YMMWORD PTR [rsi+rax]
        vfmadd231pd     ymm0, ymm6, YMMWORD PTR [rdi+rax]
        add     rax, 32
.L32:
        vmovupd ymm5, YMMWORD PTR [rsi+rax]
        vfmadd231pd     ymm0, ymm5, YMMWORD PTR [rdi+rax]
        add     rax, 32
.L31:
        vmovupd ymm1, YMMWORD PTR [rsi+rax]
        vfmadd231pd     ymm0, ymm1, YMMWORD PTR [rdi+rax]
        add     rax, 32
.L30:
        vmovupd ymm2, YMMWORD PTR [rsi+rax]
        vfmadd231pd     ymm0, ymm2, YMMWORD PTR [rdi+rax]
        add     rax, 32
        cmp     rax, r8
        je      .L44
.L4:
        vmovupd ymm8, YMMWORD PTR [rsi+rax]
        vmovupd ymm9, YMMWORD PTR [rsi+32+rax]
        vfmadd231pd     ymm0, ymm8, YMMWORD PTR [rdi+rax]
        vmovupd ymm10, YMMWORD PTR [rsi+64+rax]
        vmovupd ymm11, YMMWORD PTR [rsi+96+rax]
        vmovupd ymm12, YMMWORD PTR [rsi+128+rax]
        vmovupd ymm13, YMMWORD PTR [rsi+160+rax]
        vfmadd231pd     ymm0, ymm9, YMMWORD PTR [rdi+32+rax]
        vmovupd ymm14, YMMWORD PTR [rsi+192+rax]
        vmovupd ymm15, YMMWORD PTR [rsi+224+rax]
        vfmadd231pd     ymm0, ymm10, YMMWORD PTR [rdi+64+rax]
        vfmadd231pd     ymm0, ymm11, YMMWORD PTR [rdi+96+rax]
        vfmadd231pd     ymm0, ymm12, YMMWORD PTR [rdi+128+rax]
        vfmadd231pd     ymm0, ymm13, YMMWORD PTR [rdi+160+rax]
        vfmadd231pd     ymm0, ymm14, YMMWORD PTR [rdi+192+rax]
        vfmadd231pd     ymm0, ymm15, YMMWORD PTR [rdi+224+rax]
        add     rax, 256
        cmp     rax, r8
        jne     .L4
.L44:
        vmovapd xmm3, xmm0
        vextractf64x2   xmm0, ymm0, 0x1
        vaddpd  xmm4, xmm3, xmm0
        mov     r9, rcx
        and     r9, -2
        and     ecx, 1
        vmovsd  xmm2, xmm4, xmm4
        vunpckhpd       xmm0, xmm4, xmm4
        je      .L48
        vzeroupper
.L3:
        sal     r9, 4
        vmovupd xmm6, XMMWORD PTR [rdi+r9]
        vunpcklpd       xmm7, xmm2, xmm0
        vfmadd132pd     xmm6, xmm7, XMMWORD PTR [rsi+r9]
        vmovsd  xmm2, xmm6, xmm6
        vunpckhpd       xmm0, xmm6, xmm6
        vaddsd  xmm0, xmm0, xmm2
        ret
.L48:
        vzeroupper
        vaddsd  xmm0, xmm0, xmm2
        ret
.L47:
        vmovupd ymm3, YMMWORD PTR [rsi]
        mov     eax, 32
        vfmadd231pd     ymm0, ymm3, YMMWORD PTR [rdi]
        jmp     .L35
.L6:
        vxorpd  xmm0, xmm0, xmm0
        ret
.L7:
        vxorpd  xmm0, xmm0, xmm0
        xor     r9d, r9d
        vmovsd  xmm2, xmm0, xmm0
        jmp     .L3

Compilateur Clang : 
Premier Code : 
-O1 : 
dotprod:                                # @dotprod
        xorpd   xmm0, xmm0
        test    rdx, rdx
        je      .LBB0_3
        xor     eax, eax
.LBB0_2:                                # =>This Inner Loop Header: Depth=1
        movsd   xmm1, qword ptr [rdi + 8*rax]   # xmm1 = mem[0],zero
        mulsd   xmm1, qword ptr [rsi + 8*rax]
        addsd   xmm0, xmm1
        add     rax, 1
        cmp     rdx, rax
        jne     .LBB0_2
.LBB0_3:
        ret


-O2 : 
dotprod:                                # @dotprod
        test    rdx, rdx
        je      .LBB0_1
        lea     rcx, [rdx - 1]
        mov     eax, edx
        and     eax, 3
        cmp     rcx, 3
        jae     .LBB0_8
        xorpd   xmm0, xmm0
        xor     ecx, ecx
        jmp     .LBB0_4
.LBB0_1:
        xorps   xmm0, xmm0
        ret
.LBB0_8:
        and     rdx, -4
        xorpd   xmm0, xmm0
        xor     ecx, ecx
.LBB0_9:                                # =>This Inner Loop Header: Depth=1
        movsd   xmm1, qword ptr [rdi + 8*rcx]   # xmm1 = mem[0],zero
        movsd   xmm2, qword ptr [rdi + 8*rcx + 8] # xmm2 = mem[0],zero
        mulsd   xmm1, qword ptr [rsi + 8*rcx]
        mulsd   xmm2, qword ptr [rsi + 8*rcx + 8]
        addsd   xmm1, xmm0
        movsd   xmm3, qword ptr [rdi + 8*rcx + 16] # xmm3 = mem[0],zero
        mulsd   xmm3, qword ptr [rsi + 8*rcx + 16]
        addsd   xmm2, xmm1
        movsd   xmm0, qword ptr [rdi + 8*rcx + 24] # xmm0 = mem[0],zero
        mulsd   xmm0, qword ptr [rsi + 8*rcx + 24]
        addsd   xmm3, xmm2
        addsd   xmm0, xmm3
        add     rcx, 4
        cmp     rdx, rcx
        jne     .LBB0_9
.LBB0_4:
        test    rax, rax
        je      .LBB0_7
        lea     rdx, [rsi + 8*rcx]
        lea     rcx, [rdi + 8*rcx]
        xor     esi, esi
.LBB0_6:                                # =>This Inner Loop Header: Depth=1
        movsd   xmm1, qword ptr [rcx + 8*rsi]   # xmm1 = mem[0],zero
        mulsd   xmm1, qword ptr [rdx + 8*rsi]
        addsd   xmm0, xmm1
        add     rsi, 1
        cmp     rax, rsi
        jne     .LBB0_6
.LBB0_7:
        ret

-O3 :
dotprod:                                # @dotprod
        test    rdx, rdx
        je      .LBB0_1
        lea     rcx, [rdx - 1]
        mov     eax, edx
        and     eax, 3
        cmp     rcx, 3
        jae     .LBB0_8
        xorpd   xmm0, xmm0
        xor     ecx, ecx
        jmp     .LBB0_4
.LBB0_1:
        xorps   xmm0, xmm0
        ret
.LBB0_8:
        and     rdx, -4
        xorpd   xmm0, xmm0
        xor     ecx, ecx
.LBB0_9:                                # =>This Inner Loop Header: Depth=1
        movsd   xmm1, qword ptr [rdi + 8*rcx]   # xmm1 = mem[0],zero
        movsd   xmm2, qword ptr [rdi + 8*rcx + 8] # xmm2 = mem[0],zero
        mulsd   xmm1, qword ptr [rsi + 8*rcx]
        mulsd   xmm2, qword ptr [rsi + 8*rcx + 8]
        addsd   xmm1, xmm0
        movsd   xmm3, qword ptr [rdi + 8*rcx + 16] # xmm3 = mem[0],zero
        mulsd   xmm3, qword ptr [rsi + 8*rcx + 16]
        addsd   xmm2, xmm1
        movsd   xmm0, qword ptr [rdi + 8*rcx + 24] # xmm0 = mem[0],zero
        mulsd   xmm0, qword ptr [rsi + 8*rcx + 24]
        addsd   xmm3, xmm2
        addsd   xmm0, xmm3
        add     rcx, 4
        cmp     rdx, rcx
        jne     .LBB0_9
.LBB0_4:
        test    rax, rax
        je      .LBB0_7
        lea     rdx, [rsi + 8*rcx]
        lea     rcx, [rdi + 8*rcx]
        xor     esi, esi
.LBB0_6:                                # =>This Inner Loop Header: Depth=1
        movsd   xmm1, qword ptr [rcx + 8*rsi]   # xmm1 = mem[0],zero
        mulsd   xmm1, qword ptr [rdx + 8*rsi]
        addsd   xmm0, xmm1
        add     rsi, 1
        cmp     rax, rsi
        jne     .LBB0_6
.LBB0_7:
        ret

-Ofast :
dotprod:                                # @dotprod
        test    rdx, rdx
        je      .LBB0_1
        cmp     rdx, 3
        ja      .LBB0_4
        xorpd   xmm0, xmm0
        xor     eax, eax
        jmp     .LBB0_11
.LBB0_1:
        xorps   xmm0, xmm0
        ret
.LBB0_4:
        mov     rax, rdx
        and     rax, -4
        lea     rcx, [rax - 4]
        mov     r8, rcx
        shr     r8, 2
        add     r8, 1
        test    rcx, rcx
        je      .LBB0_5
        mov     r9, r8
        and     r9, -2
        neg     r9
        xorpd   xmm1, xmm1
        xor     ecx, ecx
        xorpd   xmm0, xmm0
.LBB0_7:                                # =>This Inner Loop Header: Depth=1
        movupd  xmm2, xmmword ptr [rdi + 8*rcx]
        movupd  xmm3, xmmword ptr [rdi + 8*rcx + 16]
        movupd  xmm4, xmmword ptr [rdi + 8*rcx + 32]
        movupd  xmm5, xmmword ptr [rdi + 8*rcx + 48]
        movupd  xmm6, xmmword ptr [rsi + 8*rcx]
        mulpd   xmm6, xmm2
        addpd   xmm6, xmm1
        movupd  xmm2, xmmword ptr [rsi + 8*rcx + 16]
        mulpd   xmm2, xmm3
        addpd   xmm2, xmm0
        movupd  xmm1, xmmword ptr [rsi + 8*rcx + 32]
        mulpd   xmm1, xmm4
        addpd   xmm1, xmm6
        movupd  xmm0, xmmword ptr [rsi + 8*rcx + 48]
        mulpd   xmm0, xmm5
        addpd   xmm0, xmm2
        add     rcx, 8
        add     r9, 2
        jne     .LBB0_7
        test    r8b, 1
        je      .LBB0_10
.LBB0_9:
        movupd  xmm2, xmmword ptr [rsi + 8*rcx]
        movupd  xmm3, xmmword ptr [rsi + 8*rcx + 16]
        movupd  xmm4, xmmword ptr [rdi + 8*rcx]
        mulpd   xmm4, xmm2
        addpd   xmm1, xmm4
        movupd  xmm2, xmmword ptr [rdi + 8*rcx + 16]
        mulpd   xmm2, xmm3
        addpd   xmm0, xmm2
.LBB0_10:
        addpd   xmm1, xmm0
        movapd  xmm0, xmm1
        unpckhpd        xmm0, xmm1                      # xmm0 = xmm0[1],xmm1[1]
        addsd   xmm0, xmm1
        cmp     rax, rdx
        je      .LBB0_12
.LBB0_11:                               # =>This Inner Loop Header: Depth=1
        movsd   xmm1, qword ptr [rsi + 8*rax]   # xmm1 = mem[0],zero
        mulsd   xmm1, qword ptr [rdi + 8*rax]
        addsd   xmm0, xmm1
        add     rax, 1
        cmp     rdx, rax
        jne     .LBB0_11
.LBB0_12:
        ret
.LBB0_5:
        xorpd   xmm1, xmm1
        xor     ecx, ecx
        xorpd   xmm0, xmm0
        test    r8b, 1
        jne     .LBB0_9
        jmp     .LBB0_10

kamikaze:
dotprod:                                # @dotprod
        test    rdx, rdx
        je      .LBB0_1
        cmp     rdx, 15
        ja      .LBB0_4
        vxorpd  xmm0, xmm0, xmm0
        xor     eax, eax
        jmp     .LBB0_11
.LBB0_1:
        vxorps  xmm0, xmm0, xmm0
        ret
.LBB0_4:
        mov     rax, rdx
        and     rax, -16
        lea     rcx, [rax - 16]
        mov     r8, rcx
        shr     r8, 4
        inc     r8
        test    rcx, rcx
        je      .LBB0_5
        mov     r9, r8
        and     r9, -2
        neg     r9
        vxorpd  xmm0, xmm0, xmm0
        xor     ecx, ecx
        vxorpd  xmm1, xmm1, xmm1
        vxorpd  xmm2, xmm2, xmm2
        vxorpd  xmm3, xmm3, xmm3
.LBB0_7:                                # =>This Inner Loop Header: Depth=1
        vmovupd ymm4, ymmword ptr [rsi + 8*rcx]
        vmovupd ymm5, ymmword ptr [rsi + 8*rcx + 32]
        vmovupd ymm6, ymmword ptr [rsi + 8*rcx + 64]
        vmovupd ymm7, ymmword ptr [rsi + 8*rcx + 96]
        vfmadd132pd     ymm4, ymm0, ymmword ptr [rdi + 8*rcx] # ymm4 = (ymm4 * mem) + ymm0
        vfmadd132pd     ymm5, ymm1, ymmword ptr [rdi + 8*rcx + 32] # ymm5 = (ymm5 * mem) + ymm1
        vfmadd132pd     ymm6, ymm2, ymmword ptr [rdi + 8*rcx + 64] # ymm6 = (ymm6 * mem) + ymm2
        vfmadd132pd     ymm7, ymm3, ymmword ptr [rdi + 8*rcx + 96] # ymm7 = (ymm7 * mem) + ymm3
        vmovupd ymm0, ymmword ptr [rsi + 8*rcx + 128]
        vmovupd ymm1, ymmword ptr [rsi + 8*rcx + 160]
        vmovupd ymm2, ymmword ptr [rsi + 8*rcx + 192]
        vmovupd ymm3, ymmword ptr [rsi + 8*rcx + 224]
        vfmadd132pd     ymm0, ymm4, ymmword ptr [rdi + 8*rcx + 128] # ymm0 = (ymm0 * mem) + ymm4
        vfmadd132pd     ymm1, ymm5, ymmword ptr [rdi + 8*rcx + 160] # ymm1 = (ymm1 * mem) + ymm5
        vfmadd132pd     ymm2, ymm6, ymmword ptr [rdi + 8*rcx + 192] # ymm2 = (ymm2 * mem) + ymm6
        vfmadd132pd     ymm3, ymm7, ymmword ptr [rdi + 8*rcx + 224] # ymm3 = (ymm3 * mem) + ymm7
        add     rcx, 32
        add     r9, 2
        jne     .LBB0_7
        test    r8b, 1
        je      .LBB0_10
.LBB0_9:
        vmovupd ymm4, ymmword ptr [rsi + 8*rcx]
        vmovupd ymm5, ymmword ptr [rsi + 8*rcx + 32]
        vmovupd ymm6, ymmword ptr [rsi + 8*rcx + 64]
        vmovupd ymm7, ymmword ptr [rsi + 8*rcx + 96]
        vfmadd231pd     ymm3, ymm7, ymmword ptr [rdi + 8*rcx + 96] # ymm3 = (ymm7 * mem) + ymm3
        vfmadd231pd     ymm2, ymm6, ymmword ptr [rdi + 8*rcx + 64] # ymm2 = (ymm6 * mem) + ymm2
        vfmadd231pd     ymm1, ymm5, ymmword ptr [rdi + 8*rcx + 32] # ymm1 = (ymm5 * mem) + ymm1
        vfmadd231pd     ymm0, ymm4, ymmword ptr [rdi + 8*rcx] # ymm0 = (ymm4 * mem) + ymm0
.LBB0_10:
        vaddpd  ymm1, ymm1, ymm3
        vaddpd  ymm0, ymm0, ymm2
        vaddpd  ymm0, ymm0, ymm1
        vextractf128    xmm1, ymm0, 1
        vaddpd  xmm0, xmm0, xmm1
        vpermilpd       xmm1, xmm0, 1           # xmm1 = xmm0[1,0]
        vaddsd  xmm0, xmm0, xmm1
        cmp     rax, rdx
        je      .LBB0_12
.LBB0_11:                               # =>This Inner Loop Header: Depth=1
        vmovsd  xmm1, qword ptr [rsi + 8*rax]   # xmm1 = mem[0],zero
        vfmadd231sd     xmm0, xmm1, qword ptr [rdi + 8*rax] # xmm0 = (xmm1 * mem) + xmm0
        inc     rax
        cmp     rdx, rax
        jne     .LBB0_11
.LBB0_12:
        vzeroupper
        ret
.LBB0_5:
        vxorpd  xmm0, xmm0, xmm0
        xor     ecx, ecx
        vxorpd  xmm1, xmm1, xmm1
        vxorpd  xmm2, xmm2, xmm2
        vxorpd  xmm3, xmm3, xmm3
        test    r8b, 1
        jne     .LBB0_9
        jmp     .LBB0_10

Deuxième Code : 
double dotprod_unroll2(double *restrict a, double *restrict b, unsigned long long n)
{
double d1 = 0.0;
double d2 = 0.0;
for (unsigned long long i = 0; i < n; i += 2)
{
d1 += (a[i] * b[i]);
d2 += (a[i + 1] * b[i + 1]);
}
return (d1 + d2);
}

-O1 : 
dotprod_unroll2:                        # @dotprod_unroll2
        test    rdx, rdx
        je      .LBB0_1
        xorpd   xmm1, xmm1
        xor     eax, eax
        xorpd   xmm0, xmm0
.LBB0_4:                                # =>This Inner Loop Header: Depth=1
        movsd   xmm2, qword ptr [rdi + 8*rax]   # xmm2 = mem[0],zero
        movsd   xmm3, qword ptr [rdi + 8*rax + 8] # xmm3 = mem[0],zero
        mulsd   xmm2, qword ptr [rsi + 8*rax]
        mulsd   xmm3, qword ptr [rsi + 8*rax + 8]
        addsd   xmm0, xmm2
        addsd   xmm1, xmm3
        add     rax, 2
        cmp     rax, rdx
        jb      .LBB0_4
        addsd   xmm0, xmm1
        ret
.LBB0_1:
        xorpd   xmm0, xmm0
        xorpd   xmm1, xmm1
        addsd   xmm0, xmm1
        ret


-O2 : 
dotprod_unroll2:                        # @dotprod_unroll2
        xorpd   xmm1, xmm1
        test    rdx, rdx
        je      .LBB0_3
        xor     eax, eax
.LBB0_2:                                # =>This Inner Loop Header: Depth=1
        movupd  xmm0, xmmword ptr [rdi + 8*rax]
        movupd  xmm2, xmmword ptr [rsi + 8*rax]
        mulpd   xmm2, xmm0
        addpd   xmm1, xmm2
        add     rax, 2
        cmp     rax, rdx
        jb      .LBB0_2
.LBB0_3:
        movapd  xmm0, xmm1
        unpckhpd        xmm0, xmm1                      # xmm0 = xmm0[1],xmm1[1]
        addsd   xmm0, xmm1
        ret

-O3 :
dotprod_unroll2:                        # @dotprod_unroll2
        xorpd   xmm1, xmm1
        test    rdx, rdx
        je      .LBB0_3
        xor     eax, eax
.LBB0_2:                                # =>This Inner Loop Header: Depth=1
        movupd  xmm0, xmmword ptr [rdi + 8*rax]
        movupd  xmm2, xmmword ptr [rsi + 8*rax]
        mulpd   xmm2, xmm0
        addpd   xmm1, xmm2
        add     rax, 2
        cmp     rax, rdx
        jb      .LBB0_2
.LBB0_3:
        movapd  xmm0, xmm1
        unpckhpd        xmm0, xmm1                      # xmm0 = xmm0[1],xmm1[1]
        addsd   xmm0, xmm1
        ret

-Ofast :
dotprod_unroll2:                        # @dotprod_unroll2
        xorpd   xmm1, xmm1
        test    rdx, rdx
        je      .LBB0_3
        xor     eax, eax
.LBB0_2:                                # =>This Inner Loop Header: Depth=1
        movupd  xmm0, xmmword ptr [rdi + 8*rax]
        movupd  xmm2, xmmword ptr [rsi + 8*rax]
        mulpd   xmm2, xmm0
        addpd   xmm1, xmm2
        add     rax, 2
        cmp     rax, rdx
        jb      .LBB0_2
.LBB0_3:
        movapd  xmm0, xmm1
        unpckhpd        xmm0, xmm1                      # xmm0 = xmm0[1],xmm1[1]
        addsd   xmm0, xmm1
        ret

kamikaze:

dotprod_unroll2:                        # @dotprod_unroll2
        vxorpd  xmm0, xmm0, xmm0
        test    rdx, rdx
        je      .LBB0_3
        xor     eax, eax
.LBB0_2:                                # =>This Inner Loop Header: Depth=1
        vmovupd xmm1, xmmword ptr [rsi + 8*rax]
        vfmadd231pd     xmm0, xmm1, xmmword ptr [rdi + 8*rax] # xmm0 = (xmm1 * mem) + xmm0
        add     rax, 2
        cmp     rax, rdx
        jb      .LBB0_2
.LBB0_3:
        vpermilpd       xmm1, xmm0, 1           # xmm1 = xmm0[1,0]
        vaddsd  xmm0, xmm1, xmm0
        ret


