# Machine-Level Programming: Basics & Control

<div style="page-break-after: always;"></div>


---

## Introduction to Machine-Level Programming

### 강의 주제

- History of Intel processors and architectures
    
- C, assembly, machine code
    
- Assembly Basics: Registers, operands, move
    
- Arithmetic & logical operations
    
<div style="page-break-after: always;"></div>

---

## Intel x86 Processor Evolution

### 주요 마일스톤

|이름|연도|트랜지스터 수|속도 (MHz)|
|---|---|---|---|
|8086|1978|29K|5–10|
|386|1985|275K|16–33|
|Pentium 4|2001|42M|~3000|
|Core i7|2008|731M|1700–3900|

- 8086: 첫 16-bit Intel 프로세서, IBM PC 기반
    
- 386: 첫 32-bit 프로세서 (IA32), UNIX 실행 가능
    
- Pentium 4E (2004): 첫 x86-64 프로세서
    
<div style="page-break-after: always;"></div>

---

## C, Assembly, Machine Code

- C → Assembly → Machine Code
    
- Assembly는 사람 친화적인 low-level 표현
    

### ISA vs. Microarchitecture

| 용어                | 설명                               |
| ----------------- | -------------------------------- |
| ISA               | 프로그래머가 보는 인터페이스 (명령어 집합, 레지스터 등) |
| Microarchitecture | 실제 구현 (파이프라인, 캐시 크기 등)           |
<div style="page-break-after: always;"></div>

---

## Assembly Basics

### Registers

- 16개의 64비트 레지스터: `%rax`, `%rbx`, `%rcx`, ..., `%r15`
    

### Operand Types

- Immediate: `$0x4`
    
- Register: `%rax`
    
- Memory: `(%rax)`, `8(%rbp)`, `(%rax,%rbx,4)`
    

### Move Instructions

- `movq Src, Dest`: 데이터 복사
    
<div style="page-break-after: always;"></div>

---

## Arithmetic & Logical Operations

### 주요 명령어

```assembly
addq %rax, %rbx   # 덧셈
subq %rbx, %rax   # 뺄셈
imulq %rcx, %rdx  # 곱셈
salq $1, %rax     # shift left
shrq $2, %rcx     # shift right
```

- 연산의 결과는 Condition Code에 영향을 미침 (다음 섹션)
    
<div style="page-break-after: always;"></div>

---

## Processor State & Condition Codes

### 주요 상태 요소

- `%rip`: 현재 명령어 포인터
    
- `%rsp`: 스택 포인터
    
- Condition Codes:
    
    - CF (Carry), ZF (Zero), SF (Sign), OF (Overflow)
        

### Condition Code 설정

- 암시적 설정: `addq`, `subq` 등 연산 후 자동 설정
    
- 명시적 설정:
    
    - `cmpq Src2, Src1` → `Src1 - Src2`와 유사
        
    - `testq Src2, Src1` → `Src1 & Src2`
        
<div style="page-break-after: always;"></div>

---

## 조건 분기 & 점프 명령

### Jump 명령

| 명령어   | 조건 (Condition)   | 의미               |
| ----- | ---------------- | ---------------- |
| `jmp` | 무조건              | 무조건 점프           |
| `je`  | ZF == 1          | Equal            |
| `jne` | ZF == 0          | Not equal        |
| `jg`  | (~(SF^OF)) & ~ZF | Greater (signed) |
| `jl`  | (SF^OF)          | Less (signed)    |
<div style="page-break-after: always;"></div>

---

## 조건부 표현식 예시

### C 코드

```c
long absdiff(long x, long y) {
  return x > y ? x - y : y - x;
}
```

### Assembly 코드 (조건 분기 사용)

```assembly
cmpq %rsi, %rdi
jle .L1
movq %rdi, %rax
subq %rsi, %rax
ret
.L1:
movq %rsi, %rax
subq %rdi, %rax
ret
```

<div style="page-break-after: always;"></div>

---

## 조건부 이동 (Conditional Move)

### 장점

- 분기(branch)를 피할 수 있음 → 파이프라인 성능 향상
    

### 예시

```assembly
movq %rdi, %rax   # x
subq %rsi, %rax   # result = x - y
movq %rsi, %rdx
subq %rdi, %rdx   # eval = y - x
cmpq %rsi, %rdi
cmovle %rdx, %rax # if (x <= y) result = eval
```

<div style="page-break-after: always;"></div>

---

## 루프 구조 (Do-While, While)

### 예제: Popcount 구현

```c
do {
  result += x & 0x1;
  x >>= 1;
} while (x);
```

### Goto 변환

```assembly
.L2:
  ...
  jne .L2
```

- while 루프는 jump-to-middle 형식으로 구현됨
    
<div style="page-break-after: always;"></div>

---

## Switch 문과 Jump Table

### C 코드

```c
switch(x) {
  case 1: w = y*z; break;
  case 2: w = y/z;
  case 3: w += z; break;
  case 5:
  case 6: w -= z; break;
  default: w = 2;
}
```

### Assembly 방식

- x 값으로 jump table의 주소를 계산해 간접 점프 수행
    
- 효율적인 분기 처리 (특히 case가 많을 때)
    

<div style="page-break-after: always;"></div>

---

## 요약 (Summary)

### C Level Control Structures

- if-else, while, do-while, for, switch
    

### Assembly Level

- 조건 점프, 조건 이동, jump table, decision tree
    

### 컴파일러 기법

- 루프는 do-while 또는 jump-to-middle 형식으로
    
- switch는 jump table 또는 decision tree로 변환됨
    
<div style="page-break-after: always;"></div>

---