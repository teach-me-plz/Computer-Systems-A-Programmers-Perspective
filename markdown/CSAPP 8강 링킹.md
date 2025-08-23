# 링킹(Linking)

## 예제 C 프로그램

```c
// main.c

int sum(int *a, int n);

int array[2] = {1, 2};

int main() {
    int val = sum(array, 2);
    return val;
}


// sum.c

int sum(int *a, int n) {
    int i, s = 0;
    for (i = 0; i < n; i++) {
        s += a[i];
    }
    return s;
}
```

---

## 정적 링크 (Static Linking)

- 프로그램은 **컴파일러 드라이버 (gcc, clang 등)** 을 사용해 번역 및 링크됨
    

```
$> gcc -Og -o prog main.c sum.c 
$> ./prog
```

- 각 `.c` 파일은 분리 컴파일되어 **재배치 가능(relocatable) 오브젝트 파일** 생성
    
- 링크 후 모든 함수 코드와 데이터가 포함된 **완전한 실행 파일(executable)** 생성
    

---

## 왜 링커(Linker)가 필요한가?

## 1. 모듈화(Modularity)

- 하나의 대형 소스 파일 대신 작고 독립적인 소스 파일 묶음으로 프로그램 작성 가능
  -> 미니쉘 파일 하나로 작성 가능? 더 커지면?
    
- 공통 함수들을 **라이브러리**로 묶어 재사용 가능
    
    - 예: 수학 라이브러리, 표준 C 라이브러리
        

## 2. 효율성(Efficiency)

- **시간 절약**: 소스 파일 일부만 수정하면 전체를 다시 컴파일할 필요 없이 링크만 다시 수행
    
- **공간 절약**: 큰 라이브러리에서 실제 사용된 함수만 실행 파일에 포함
    

---

## 링커가 하는 일

## Step 1: 심볼 해석(Symbol Resolution)

- 프로그램은 **심볼(Symbol)**(전역 변수, 함수)을 정의하거나 참조함
    
```c
void swap() { ... }   /* 심볼 정의 */
swap();               /* 심볼 참조 */
int *xp = &x;         /* xp 정의, x 참조 */
```
    
- 심볼 정의는 오브젝트 파일의 **심볼 테이블(symbol table)** 에 저장됨
    
- 링크 단계에서 각 심볼 참조를 정확히 하나의 심볼 정의와 매칭

- 지역변수는 런타임에 스택에서 관리됨(심볼 x)

## Step 2: 재배치(Relocation)

- 여러 오브젝트 파일의 코드와 데이터 세그먼트를 합침
    
- 각 심볼이 실행 파일 내 **최종 주소**로 이동
    
- 참조되는 주소도 모두 갱신
    

---

## 오브젝트 파일 종류

1. **재배치 가능 오브젝트 파일(.o)**
    
    - 여러 개 합쳐 실행 파일 생성 가능
        
2. **실행 가능한 오브젝트 파일(a.out)**
    
    - 메모리에 바로 적재되어 실행될 수 있는 형태
        
3. **공유 오브젝트 파일(.so)**
    
    - 동적 링킹(Dynamic Linking)에 사용 (로드 시점 혹은 실행 중)
        
    - 윈도우에서는 DLL(동적 링크 라이브러리)라고 부름
        

---

## ELF (Executable and Linkable Format)

- 리눅스 표준 이진 포맷
    
- **하나의 통합 포맷**으로 `.o`, 실행파일, `.so` 표현 가능
    

---

## ELF 파일 구조
<img src="https://velog.velcdn.com/images/syk25/post/b89fcaad-cd34-459f-8297-f378273edfea/image.png"/>

- **ELF 헤더**: 워드 크기, 엔디안, 파일 종류 등
    
- **세그먼트 헤더 테이블**: 메모리 매핑 정보
    
- **.text**: 코드
    
- **.rodata**: 읽기 전용 데이터
	  포맷 스트링, 스위치문 점프 테이블 등
    
- **.data**: 초기화된 전역 변수, 정적 변수
    
- **.bss**: 초기화되지 않은 전역 변수, 정적 변수 (공간 차지는 안함)
	    ex) `int arr[4200];` 이렇게 초기화하지 않은 전역변수에 대해
	    int 4200개만큼의 데이터는 목적파일에 존재하지 않음
	    실행시에 int 4200개만큼 메모리를 확보하고 0으로 채워넣는다.

참고) 0으로 초기화된 전역변수나 정적변수도 .bss섹션에 포함된다. 따로 값을 저장할 필요가 없기 때문

- **.symtab**: 심볼 테이블
		  정의/참조되는 전역변수, 함수에 대한 정보
    
- **.rel.text**: 이 목적파일을 다른 파일과 연결할 때 수정되어야 하는 .text 섹션(코드) 내 위치의 리스트
		  외부 함수를 호출하거나 전역변수를 참조하는 인스트럭션은 모두 수정되어야 함
    
- **.rel.data**: 이 모듈이 정의/참조 하는 전역변수에 대한 재배치 정보
		  전역 변수가 외부 함수의 주소를 담고 있다거나 하는 경우
    
- **.debug**: 심볼 디버깅을 위한 섹션
		  gcc -g옵션으로 생성됨
    
- **.line**: 최초 C 소스 프로그램과 .text 머신 코드간의 라인 매핑
	  이게 있어야 lldb에서 어떤 라인 실행중인지 보여주지
	  gcc -g 옵션으로 생성됨
  - **.strtab**: strtab과 debug 섹션에 쓰이는 스트링 테이블




---

## 링커 심볼 종류

- **글로벌 심볼 (Global)**: 외부 모듈에서 참조 가능한 함수/변수
    
- **외부 심볼 (External)**: 현재 모듈에서 참조하지만 정의는 다른 모듈에 존재
    
- **로컬 심볼 (Local)**: `static` 키워드로 제한된 심볼, 해당 모듈 내에서만 사용
    

---

## 심볼 충돌 해결 규칙

강한 심볼: 프로시저(함수), 초기화된 글로벌 심볼
약한 심볼: 초기화 되지 않은 글로벌 심볼


1. **강한 심볼(Strong)**은 중복될 수 없음 (중복되면 에러)
    
2. **강한 심볼 vs 약한 심볼(Weak)** → 강한 심볼이 우선
        
3. **여러 약한 심볼이 있는 경우** → 임의 하나 선택


근데 내 환경에서는 세 경우 모두 링크 안해줌
일단 책에는 저렇게 적혀있습니다..;

gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04)
GNU ld (GNU Binutils for Ubuntu) 2.38
기준


---
## 전역 변수에 대한 고찰

내 링커에서는 알아서 막아줘도 다른 링커에서도 동일할거라는 보장이 없다.

#### Nightmare scenario
1. 초기화 되지 않은 동일한 구조체 2개가
2. 각각 다른 파일에서 다른 컴파일러로 컴파일 되었고
3. 그 컴파일러의 alignment rule이 다르다면..?

이런 야랄나기 싫으면 기억할것

- 그냥 전역변수 안쓰면 됨ㅋㅋ

- 꼭 써야 한다면
	- 가능하면 static을 쓰자
	- 전역변수 선언할거면 초기화 해라
	- 다른 모듈의 전역변수 참조할거면 extern을 써라


---

## 재배치 예시

`.o` 파일에는 다음과 같은 **재배치 엔트리(Relocation Entry)** 포함

```
0000000000000000 <main>:
   0:   f3 0f 1e fa             endbr64 
   4:   48 83 ec 08             sub    $0x8,%rsp
   8:   be 02 00 00 00          mov    $0x2,%esi
   d:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi        # 14 <main+0x14>
                        10: R_X86_64_PC32       array-0x4
  14:   e8 00 00 00 00          call   19 <main+0x19>
                        15: R_X86_64_PLT32      sum-0x4
  19:   48 83 c4 08             add    $0x8,%rsp
  1d:   c3                      ret    
```

- 참조된 객체(`array`, `sum`)의 최종 실행 주소로 갱신 필요
    

```
10: R_X86_64_PC32 array  # array 주소로 바뀜
15: R_X86_64_PLT32 sum  # sum 함수 주소로 바뀜
```

---

## 실행 파일 적재

운영체제 커널이 다음을 메모리에 매핑함:

- Read-only 코드 섹션 (.text, .rodata)
    
- Read/Write 데이터 섹션 (.data, .bss)
    
- 힙(heap), 스택(stack)
    
- 공유 라이브러리

<img src="https://velog.velcdn.com/images/junttang/post/4845599c-5bcf-40f4-b7be-6dea82324c2a/image.png" />


---

## 라이브러리 패키징

- 여러 함수(예: 수학, 문자열, 입출력)를 효율적으로 제공하는 방법
    
- 옛날 방식: **정적 라이브러리(.a, archive)**
    
    - 여러 `.o` 파일을 묶어 하나의 아카이브 생성
        
    - 필요할 때만 객체 포함
        
- 현대 방식: **공유 라이브러리(.so)**
    
    - 실행 파일 간 코드 중복 제거
        
    - 버그 수정 시 재컴파일 불필요
        

---

## 정적 라이브러리 사용법


```bash
// 생성
ar cr libft.a *.o

// 직접 지정
cc -o client client.o ./libft/libft.a

// 또는
cc -o client client.o -L. -lft
```

- 링커는 커맨드 라인에 등장한 `.o`와 `.a`를 순서대로 검색
    
- 순서가 중요하며, 보통 라이브러리는 **맨 끝**에 넣어야 함

- -L. -> 현재 디렉터리를 라이브러리 검색 경로에 추가
  `-l<name>` -> `lib<name>.a 또는 lib<name>.so`를 찾음

<img src="https://i.sstatic.net/85V5p.png" />

```
// nm libft.a 중 일부

ft_split.o:
                 U free
0000000000000170 t ft_free_null
00000000000001df t ft_len_until_c
0000000000000000 T ft_split
                 U ft_strlcpy
                 U malloc
000000000000022b t word_counter
```

링커는 미해석 심볼을 아카이브 멤버에 의해 정의된 심볼과 매칭하려고 시도한다.

---

## 동적 링크 (Shared Library)

- 실행 파일이 **로드될 때(load-time)** 동적 링커(`ld-linux.so`)가 `.so` 로드
    
- 실행 중에 **런타임(run-time)**에도 가능 → `dlopen()`, `dlsym()`, `dlclose()` 사용

<img src="https://i.imgur.com/Xf6uoxL.png" />


---

## 링킹 요약
- 링킹은 프로그램이 여러 오브젝트 파일로 구성될 수 있도록 하는 것

- 링킹은 프로그램의 여러 단계에서 일어날 수 있다
	- 컴파일 타임
	- 로드 타임
	- 런타임

- 링킹에 대해 이해하면 요상한 에러를 피할 수 있고, 더 나은 프로그래머가 될 수 있다.
  
  <그날 주제>를 이해하면 더 나은 프로그래머 어쩌고 이거 챕터별 템플릿인듯 ㅋㅋ

---

# 인터포지셔닝 (Interpositioning)

- 프로그램의 함수 호출을 가로채는 기법
    
- 시점:
    
    - **컴파일 타임**: 매크로 치환
        
    - **링크 타임**: `--wrap` 옵션 사용
        
    - **실행 타임**: `LD_PRELOAD` 사용
        

---

## 활용 예시

- **보안**: 샌드박싱, 암호화 후킹
  -> 반대로 입력 감시도 가능할 것 같은데?!
    
- **디버깅**: 함수 호출 추적, malloc/free 동작 확인
    
- **모니터링/프로파일링**: 함수 호출 횟수 세기, 메모리 누수 탐지
    

---

## 인터포지셔닝 방법별 요약

- **컴파일타임**: `#define malloc mymalloc` 같은 매크로 확장
    
- **링크타임**: 링커 옵션 `--wrap=malloc` 사용 → `__wrap_malloc` 호출
    
- **런타임**: `LD_PRELOAD`로 내가 작성한 `.so`를 먼저 로드하여 기존 함수 가로채기
    

---


## 제가 한번 써봤습니다
제일 간단한 링크 타임 인터포지셔닝 예시

```c
// main.c

#include <stdio.h>
#include <stdlib.h>

int main(void) {

  char *p = malloc(10);
  if (p)
    printf("malloc succeed!\n");
  else
    printf("malloc failed!\n");
}

// my_malloc.c

#include <stdlib.h>
// 항상 실패하는 malloc ㅋㅋ
void *__wrap_malloc(size_t size) { 
	return 0; 
}
```

```
cc main.c
./a.out
malloc succeed!


cc main.c my_malloc.c -Wl,--wrap=malloc  // 링크 옵션 추가
./a.out
malloc failed!
```

사실 `cc main.o my_mallo.o -Wl ...` 이렇게 재배치 가능 목적 파일 (.o 파일)을 링크할 때 쓰는 옵션이긴 한데 예시처럼 처음부터 옵션 줘도 되더라고


## ELF 재배치 정보도 맛만 보자면
`readelf -r a.out` 을 해보았다.
#### main.c 만 단독으로 컴파일 한 경우
```
Relocation section '.rela.plt' at offset 0x630 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000003fc8  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
000000003fd0  000500000007 R_X86_64_JUMP_SLO 0000000000000000 malloc@GLIBC_2.2.5 + 0
```

puts - printf()
malloc - malloc()
동적 링킹을 위해 재배치 테이블에 두 개를 기록해둔 모습

#### malloc을 가로채서 링크한 경우

```
Relocation section '.rela.plt' at offset 0x610 contains 1 entry:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000003fd0  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
```

malloc이 재배치 테이블에서 사라졌다.

왜? 링크 시점에 glibc로 가는 malloc 심볼을 my_malloc으로 바꿨고, 그 결과 동적으로 링크할 필요가 없어졌기 때문


## 결론
libft 과제는 신이다
