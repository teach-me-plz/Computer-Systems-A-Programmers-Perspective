## 1. 복습
###  메모리 계층 구조의 개요

- 빠르고 작은 고비용 메모리 → 느리고 큰 저비용 메모리로 구성
    
- 계층 (빠름 → 느림):
    
    - CPU 레지스터
        
    - L1 캐시 (SRAM)
        
    - L2 캐시 (SRAM)
        
    - L3 캐시 (SRAM)
        
    - 주메모리 DRAM
        
    - 로컬 디스크
        
    - 원격 저장소 (웹서버 등)
        
<img src="https://www.cs.swarthmore.edu/~kwebb/cs31/f18/memhierarchy/figures/MemoryHierarchy.png">

---

## 2. 캐시 메모리 기본 구조

###  정의

- SRAM 기반의 소형, 고속 메모리
    
- 하드웨어에 의해 자동으로 관리
    
- 자주 참조되는 메모리 블록을 저장 (temporal locality 활용)


###  캐시 구성 요소 (3요소 구조)

- **S (Set)**: 세트 수 (2^s)
    
- **E (Lines per set)**: 각 세트 내 블록 수 (2^e)
    
- **B (Block size)**: 블록 당 바이트 수 (2^b)
    
- 캐시 크기 C = S * E * B
    
<img src="https://velog.velcdn.com/images/msung99/post/571586b1-3c34-4ae6-a4f2-a862ec61f7b9/image.png">
###  주소 분해 방식

- Tag / Set Index / Block Offset
- 0xAB / CD / EFGH
- 태그, 인덱스, 오프셋 -> 인덱스, 오프셋
  0x....101.... -> 111 101 111
             101 101 111
    
<img src="https://cdn.jsdelivr.net/gh/jaehyeon48/jaehyeon48.github.io@main/assets/images/computer-architecture/cache-memory/memory-address.png">

###  동작 방식

1. 주소로부터 Set 선택
    
2. 해당 Set 내 Line 중 Tag 비교
    
3. 유효(valid)하면 **hit**, 없으면 **miss**
    

---

## 3. 캐시 매핑 방식

###  직접 매핑 (Direct Mapped)

- E = 1
    
- 각 세트에 1개의 블록만 저장
    
- tag가 다르면 기존 데이터 제거 후 새 데이터 적재 (eviction)
    

###  집합결합성 (Set Associative)

- E > 1 (ex: 2-way, 4-way)
    
- 한 세트에 여러 라인, 각 라인에 Tag 저장 -> 마찬가지
    
- hit 발생 확률 증가
    
- 교체 정책 필요 (LRU, LFU, 걍 랜덤 선택) -> 시간적인 자원 소모
    

###  완전결합성 (Fully Associative)

- 하나의 세트에 모든 라인이 존재 (S=1, E=전체 블록 수)
  
- 이 구조에서는 인덱스 비트가 필요없다! (주소를 Tag 비트랑 Block offset으로만 해석 )
    
- 가장 유연하지만 탐색 시간 증가 (태그를 병렬로 검색해야함)
	
-> TLB같은 작은 캐시에서만 사용하는 편 
%%(TLB는 9챕터에 나온다고함.. 저는 몰라요)%%

---

## 4. 캐시 쓰기 정책

load

store
###  Write Hit

- **Write-through**: 캐시 + 메모리 동시 갱신
    
- **Write-back**: 캐시에만 쓰고, 나중에 메모리에 반영 (dirty bit 필요)
  -> 메모리에 있는 데이터가 과연 최신 데이터인가?
  -> flush
  
	  이게 동시성 프로그램이 아니어도 발생 가능하긴 함
	  언제:
	  데이터가 코드가 될 수 있고 역도 성립하기 때문.
    

###  Write Miss

- **Write-allocate**: 캐시에 불러온 뒤 작성 (대부분 사용)
    
- **No-write-allocate**: 캐시에 안 올리고 메모리에만 기록

교수님의 제안: Write-back, Write-allocate 캐시를 가정하고 프로그래밍 하도록

---

## 5. 성능 지표

###  캐시 성능 3요소

- **Hit Time**: hit일 때 소요 시간
    
- **Miss Rate**: miss 발생 비율
    
- **Miss Penalty**: miss일 때 추가 소요 시간 (ex: DRAM 접근 100사이클 이상)
    

###  예시: 99% vs 97% hit rate
캐시 계층 1 / 메인메모리

캐시 히트의 경우에  1 사이클이 걸리고,
캐시 미스의 경우에 100 사이클이 걸린다고 가정시

평균 사이클 계산
- 97%: 1사이클 + 0.03 * 100사이클 = 4사이클
    
- 99%: 1사이클 + 0.01 * 100사이클 = 2사이클

2배 차이가 난다.


### 인텔 Core i7 캐시 계층구조

| 계층    | 용도      | 캐시 사이즈 (C) | 연관도 (Way) | 접근 시간 (사이클) | 블록 사이즈 | Sets (S) | 비고         |
| ----- | ------- | ---------- | --------- | ----------- | ------ | -------- | ---------- |
| L1 캐시 | i-/d-분리 | 32KB       | 8-way     | 4 사이클       | 64 B   | 64       | 명령어/데이터 분리 |
| L2 캐시 | 통합      | 256KB      | 8-way     | 10 사이클      | 64 B   | 64       | 코어별        |
| L3 캐시 | 통합      | 8MB        | 16-way    | 40~75 사이클   | 64 B   | 512      | 모든 코어가 공유  |


---

## 6. 캐시 친화적 코드 작성

###  Locality 원칙

- **Spatial locality (공간 지역성)**: 인접한 메모리 접근 (ex: 배열 순회)
    
- **Temporal locality (시간 지역성)**: 동일한 데이터를 반복 참조
    

###  좋은 패턴

- stride-1 순회
  -> 연속 접근
  -> 한칸씩 접근 : 63번 히트
  -> 4칸씩 접근 : 16번 말고는 다 미스
    
- inner loop에서 집중된 참조
	-> 열 우선 순회 이런거 하지말기


---

## 7. 메모리 마운틴

- stride, 배열 크기에 따른 read throughput 측정 (MB/s)
    
- 공간/시간 지역성 변화에 따른 성능 관찰
    

###  코드 (C)

```c
long data[MAXELEMS];
int test(int elems, int stride) {
    long i, sx2 = stride*2, sx3 = stride*3, sx4 = stride*4;
    long acc0 = 0, acc1 = 0, acc2 = 0, acc3 = 0;
    long length = elems, limit = length - sx4;
    for (i = 0; i < limit; i += sx4) {
        acc0 += data[i];
        acc1 += data[i+stride];
        acc2 += data[i+sx2];
        acc3 += data[i+sx3];
    }
    for (; i < length; i++) acc0 += data[i];
    return (int)((acc0 + acc1) + (acc2 + acc3));
}
```

%%3D로 보여드림 ㄱㄷㄱㄷ%%

시간 지역성, 공간 지역성을 확인할 수 있음

---

## 8. 행렬 곱셈과 캐시 성능

###  기본 ijk 버전 (C)

```c
for (i=0; i<n; i++)
  for (j=0; j<n; j++) {
    sum = 0.0;
    for (k=0; k<n; k++)
      sum += a[i][k] * b[k][j];
    c[i][j] = sum;
  }
```


###  분석
캐시에는 배열의 요소가 4개 들어간다고 가정
stride 1일일때 n-1 히트 한번 미스

첫번째는 무조건 미스

캐시 x
i: 4 미스 -> 0번째 1번째 2 3 
i:  히트



- 배열은 row-major 저장 (C에서)
    
- `a[i][k]`: stride-1 ok?
    
- `b[k][j]`: stride-N (공간 지역성 없음)
    
- `c[i][j]`: stride-1 접근 가능


### kij 버전
``` c
for (k = 0; k < n; k++)
	for (i = 0; i < n; i++) {
		r = A[i][k];
		for (j = 0; j < n; j++)
			c[i][j] += r*B[k][j];
	}
```

점근적 분석으로는 O(n^3)로 동일하지만 실행 결과도 동일할까?
###  순서에 따른 miss 비교


| 루프 순서 | a miss | b miss | c miss | 총 miss |
| ----- | ------ | ------ | ------ | ------ |
| ijk   | 0.25   | 1.00   | 0.00   | 1.25   |
| kij   | 0.00   | 0.25   | 0.25   | 0.50   |
| jki   | 1.00   | 0.00   | 1.00   | 2.00   |

### 결과
- double 행렬의 크기가 큰 경우 방법에 따라 최대 40배 가량의 속도 차이가 났음
- 이 실험의 경우에는 메모리 접근 횟수보다도 미스 비율이 더 중요하게 작용
  (ijk는 로드 2회 / kij는 로드 2회, 스토어 1회)

---

## 9. 블록화(Blocking) 기법
L1 캐시에 하나의 덩어리를 로드하고 이 덩어리에 수행해야 하는 읽기 쓰기를 전부 한다.
그리고 이 덩어리는 버리고 다음 덩어리 로드

근디 블록화는 코드를 읽고 이해하는걸 매우 어렵게 만든다..
```c
for (i=0; i<n; i+=B)
  for (j=0; j<n; j+=B)
    for (k=0; k<n; k+=B)
      for (i1=i; i1<i+B; i1++)
        for (j1=j; j1<j+B; j1++)
          for (k1=k; k1<k+B; k1++)
            c[i1*n+j1] += a[i1*n + k1] * b[k1*n + j1]; // 구경만 하세요..^^
```

자주 실행되는 라이브러리 루틴이나 최적화 컴파일러에 적합



---

## 10. 요약

###  세 줄 요약

- 캐시는 빠르지만 작다 → 그니까 작은 캐시를 활용 잘 해야함
    
- 캐시 구조 (S, E, B)와 정책 이해는 성능 최적화에 중요
    
- stride-1, ~~blocking~~, inner loop 집중 등으로 캐시 활용 극대화
  -> 블록화는 성능 - 가독성 트레이드오프 잘 고려하는게 좋지 않을까?
