## Machine-Level Programming III & IV: Procedures and Data
아래의 코드를 c언어로 다시 작성하시오. ㅋㅋ
```asembly
function:
	pushq	%rbx
	movq	%rdi, %rbx
	movl	$1, %eax
	cmpq	$1, $rdi
	jle		.L35
	leaq	-1(%rdi), %rdi
	call	function
	imulq	%rbx, %rax
.L35:
	popq	%rbx
	ret
```

<div style="page-break-after: always;"></div>

### 오늘의 주제

- Procedure의 호출과 반환 메커니즘
- 스택 구조와 호출 규약 (Calling Convention)
- 배열과 구조체의 메모리 표현
- 다차원 및 다중 수준 배열
- 정렬과 패딩을 포함한 데이터 정렬 (Alignment)
- 부동소수점(Floating Point) 기초

<div style="page-break-after: always;"></div>

## Machine-Level Programming III: Procedures

### Procedure 메커니즘

- 제어 흐름 전달: 호출 시 procedure 시작 위치로 이동, 종료 시 반환 주소로 복귀
- 데이터 전달: 인자(argument), 반환값(return value)
- 메모리 관리: procedure 실행 중에 할당, 종료 후 해제

### 예시 코드

```c
int Q(int i) {
    int t = 3 * i;
    int v[10];
    return v[t];
}
```
<div style="page-break-after: always;"></div>

### Stack 구조 (x86-64 기준)

- 스택은 메모리의 높은 주소에서 낮은 주소로 성장
- %rsp 레지스터가 stack의 top을 가리킴

```asm
pushq %rbx    # %rsp -= 8; [rsp] = %rbx
popq %rbx     # %rbx = [rsp]; %rsp += 8
```
<div style="page-break-after: always;"></div>

### Procedure Control Flow

```c
long mult2(long a, long b) {
    return a * b;
}

void multstore(long x, long y, long *dest) {
    long t = mult2(x, y);
    *dest = t;
}
```

```asm
mult2:
    mov %rdi, %rax
    imul %rsi, %rax
    retq

multstore:
    push %rbx
    mov %rdx, %rbx
    call mult2
    mov %rax, (%rbx)
    pop %rbx
    retq
```
<div style="page-break-after: always;"></div>

### Register 사용 규약

- 인자 전달: %rdi, %rsi, %rdx, %rcx, %r8, %r9
- 반환값: %rax
- Caller-saved: %rax, %rdi\~%r11
- Callee-saved: %rbx, %rbp, %r12\~%r15
<div style="page-break-after: always;"></div>

### Stack Frame 구성

- 반환 주소
- 저장된 레지스터
- 지역 변수
- 다음 호출을 위한 argument build 영역
<div style="page-break-after: always;"></div>

### 재귀 예시: popcount

```c
long pcount_r(unsigned long x) {
    if (x == 0) return 0;
    else return (x & 1) + pcount_r(x >> 1);
}
```

```asm
pcount_r:
    movl $0, %eax
    testq %rdi, %rdi
    je .L6
    pushq %rbx
    movq %rdi, %rbx
    andl $1, %ebx
    shrq %rdi
    call pcount_r
    addq %rbx, %rax
    popq %rbx
.L6:
    ret
```
<div style="page-break-after: always;"></div>

## Machine-Level Programming IV: Data

### 배열의 메모리 할당

- 선언: `T A[L];` → T 타입의 L개 원소로 구성된 배열
- 메모리 상에는 `L * sizeof(T)` 만큼의 연속된 공간이 할당됨

```c
char string[12];
int val[5];
double a[3];
char *p[3];
```
<div style="page-break-after: always;"></div>

### 배열과 포인터

- 배열 이름은 배열의 첫 번째 원소의 포인터로 취급됨
- 예: `val[i]`는 `*(val + i)` 와 동일

```c
int val[5] = {1, 2, 3, 4, 5};
val[2]; // *(val + 2)
&val[2]; // val + 2
```
<div style="page-break-after: always;"></div>

### 배열 접근 예시

```c
#define SUBJECTS 5
typedef int grade[SUBJECTS];

grade tom = {1, 5, 2, 1, 3};

int math_grade = grade[1];
```

<div style="page-break-after: always;"></div>

### 배열 원소 접근 함수

```c
int get_grade(grade g, int subject) {
    return g[subject];
}
```

```asm
movl (% rdi, %rsi, 4), %eax  # z[digit]
```
<div style="page-break-after: always;"></div>

### 배열 루프

```c
void grade_inc(grade g) {
    for (size_t i = 0; i < SUBJECTS; i++)
        g[i]++;
}
```

```asm
movl $0, %eax
.L3:
    cmpq $4, %rax
    jg .Lend
    addl $1, (%rdi,%rax,4)
    incq %rax
    jmp .L3
.Lend:
    ret
```
<div style="page-break-after: always;"></div>

### 중첩 배열 (2차원 배열)

```c
int A[R][C];
```

- Row-major order: 행 단위로 메모리에 저장됨
- 원소 접근: `A[i][j] = *(A + i*C + j)`
<div style="page-break-after: always;"></div>

### 중첩 배열 예시

```c
#define STUDENTS 4
grade class_1_grade[STUDENTS] = {
  {1,5,2,0,6},
  {1,5,2,1,3},
  {1,5,2,1,7},
  {1,5,2,2,1}
};
```
<div style="page-break-after: always;"></div>

### 중첩 배열 행 벡터 접근

```c
int* get_grade(int index) {
    return class_1_grade[index];
}
```

```asm
leaq (%rdi, %rdi, 4), %rax   # rdi * 5
leaq pgh(, %rax, 4), %rax    # 주소 계산
```
<div style="page-break-after: always;"></div>

### 중첩 배열 원소 접근

```c
int get_subject_grade(int index, int subject) {
    return class_1_grade[index][subject];
}
```

```asm
leaq (%rdi,%rdi,4), %rax
addl %rax, %rsi
movl pgh(,%rsi,4), %eax
```
<div style="page-break-after: always;"></div>

### 다중 수준 배열 (배열의 포인터)

```c

int lee[5] = {1, 5, 2, 1, 3};
int choi[5] = {0, 2, 1, 3, 9};
int park[5] = {9, 4, 7, 2, 0};

int *three_people_grade[3] = {lee, choi, park};
int get_subject_grade(size_t index, size_t subject) {
    return three_people_grade[index][subject];
}
```

```asm
salq $2, %rsi
addq univ(,%rdi,8), %rsi
movl (%rsi), %eax
```

- `univ[i][j]`는 두 번의 메모리 접근이 필요함
- `Mem[Mem[univ + 8*index] + 4*digit]`
<div style="page-break-after: always;"></div>

### 배열 vs 포인터 차이

- 중첩 배열: 연속된 메모리
- 다중 수준 배열: 각각 독립된 배열을 가리키는 포인터

```c
int class_1_grade[4][5] = {
  {1,5,2,0,6},
  {1,5,2,1,3},
  {1,5,2,1,7},
  {1,5,2,2,1}
};

// 다중 수준 배열
int *three_people_grade[3] = {lee, choi, park}
```


둘 다 `arr[i][j]`로 값을 가져올 수 있지만 내부 동작은 전혀 다름

둘 다 `arr[i][j]`의 값을 가져온다 할 때
- 중첩 배열: `Mem[arr + (20 * i) + (4 * j)]`
- 다중 수준 배열: `Mem[Mem[arr + (8 * i)] + (4 * j)]`

<div style="page-break-after: always;"></div>

### 정적 / 동적 2차원 배열 비교

```c
// 고정 크기 배열

#define N 16
typedef int fix_matrix[N][N];
int fix_ele(fix_matrix a, size_t i, size_t j) {
    return a[i][j];
}
```

```c
// 가변 길이 배열 + 직접 인덱스 계산해서 접근

#define IDX(n, i, j) ((i)*(n)+(j))
int vec_ele(size_t n, int *a, size_t i, size_t j) {
    return a[IDX(n, i, j)];
}
```

```c
// 가변 길이 배열 + 컴파일러한테 '해줘' 하기

int var_ele(size_t n, int a[n][n], size_t i, size_t j) {
    return a[i][j];
}
```

- 정적 배열: 컴파일 타임에 크기 고정
- 가변 길이 배열: C99부터 지원, 런타임에 크기 결정됨
<div style="page-break-after: always;"></div>

### 구조체 내 배열 멤버 접근

```c
struct rec {
    int a[4];
    size_t i;
    struct rec *next;
};

void set_val(struct rec *r, int val) {
    while (r) {
        int i = r->i;
        r->a[i] = val;
        r = r->next;
    }
}
```

```asm
movslq 16(%rdi), %rax      # i = r->i
movl %esi, (%rdi,%rax,4)   # r->a[i] = val
movq 24(%rdi), %rdi        # r = r->next
testq %rdi, %rdi
jne .Lloop
```
<div style="page-break-after: always;"></div>

### 배열 내 구조체, 구조체 내 배열

- 구조체 크기 = 각 멤버의 크기 + alignment를 위한 padding
- 배열 내 구조체일 경우 구조체 단위로 정렬됨

### 구조체 배열 원소 접근

```c
struct S3 { // total size - 12
	short i; 
	float v;
	short j;
} a[10];
short get_j(int idx) {
    return a[idx].j;
}
```

```asm
leaq (%rdi, %rdi, 2), %rax   # 3*idx
movzwl a+8(,%rax,4), %eax    # offset 8
```
<div style="page-break-after: always;"></div>

### 구조체 크기 최적화

```c
struct S4 { char c; int i; char d; };  // 12 bytes
struct S5 { int i; char c; char d; };  // 8 bytes
```

- 큰 타입 먼저 배치하면 padding 줄일 수 있음

<div style="page-break-after: always;"></div>

### 구조체 표현

```c
struct rec {
    int a[4];
    size_t i;
    struct rec *next;
};

int* get_ap(struct rec *r, size_t idx) {
    return &r->a[idx];
}
```

```asm
leaq (%rdi,%rsi,4), %rax
```
<div style="page-break-after: always;"></div>

### 구조체 내 정렬

```c
struct S1 {
    char c;
    int i[2];
    double v;
};
```
- 총 ?? bytes (padding 포함)
- 각 멤버는 alignment 요구를 만족해야 함
<div style="page-break-after: always;"></div>

### 최적화된 정렬

```c
struct S4 { char c; int i; char d; }; // 12 bytes
struct S5 { int i; char c; char d; }; // 8 bytes
```

큰거부터 넣으면 됨니다
<div style="page-break-after: always;"></div>

## Floating Point 기초

### 표현 방식

- IEEE 754 형식: `(-1)^s * M * 2^E`
- 정규화(normalized), 비정규화(denormalized), 특별값(NaN, ±∞) 존재
<div style="page-break-after: always;"></div>

### 레지스터 사용

- float/double 인자는 %xmm0, %xmm1 ... 로 전달

```c
float fadd(float x, float y) {
    return x + y;
}
```

```asm
addss %xmm1, %xmm0
```

```c
double dincr(double *p, double v) {
    double x = *p;
    *p = x + v;
    return x;
}
```

```asm
movsd (%rdi), %xmm0
addsd %xmm0, %xmm1
movsd %xmm1, (%rdi)
```
<div style="page-break-after: always;"></div>

### 근사 방식

- 기본: round to nearest, ties to even
- 그 외: toward zero, toward -∞, toward +∞


### 사실은...
방금 본 addss addsd 등은 SSE명령어다. 
근데 SSE가 진화한 AVX 가 나왔고 명령어 이름도 조금 바뀌고(vaddss vmovss 등)
한번에 병렬 연산할 수 있는 크기도 늘었다.

근데 궁금한 사람 없을것 같아서 넘어감 ㅅㄱ
궁금하면 책 빌려드림
<div style="page-break-after: always;"></div>

## 요약

- 우리가 지금까지 low 하다고 생각했던 C는 사실 위대한 고수준 언어였다....

- 원래 다음 챕터는 프로세서를 직접 구현하는 부분인데... 이건 강의로는 되는 챕터가 아닌 것 같아서 넘어갑니다... 

- 다음주에 프로그램 성능 최적화로 만나요~~