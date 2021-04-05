---
title: 고언어(Golang) HACKING.md 내용 정리
category: golang
tags: golang, 고언어, haking
---

Golang은 Open source project 이며 모든 개발 내역이 Github에 코드로 구현되어 있다.
하나하나의 구현 내용을 이해하고 싶다면 각 패키지를 읽어보며 이해할 수 있다.
그렇지만 방대한 양이기 때문에 처음에 접하는 경우에는 어디서부터 읽어야할지,
그리고 기본적인 구조를 어떻게 잡아야하는지 어렵기만 하다.

그렇다면 Golang의 기본적인 구조에 대해서 이해하고 싶다면 어떻게 시작해야할까?
다행히도 고언어 개발자들은 HANKING.md에 기본적인 내용을 정리해두었다.
고언어의 런타임이 어떤 구조로 이뤄져 있는지, 그리고 동시성 처리를 위한 방식이나
메모리 관리 측면에서는 어떻게 하고 있는지 등 다양한 내용이 정리되어 있다.

이번에는 HAKING.md를 번역하며 고언어에 대해서 더 깊은 이해를 해보려 한다.

원문을 읽고 싶다면 [https://golang.org/src/runtime/HACKING.md](https://golang.org/src/runtime/HACKING.md) 여기를 참고 바란다.

---

## https://golang.org/src/runtime/HACKING.md

이것은 지속적으로 업데이트되고 있는 문서이며, 때때로 업데이트 시기를 놓칠 수도 있다.
일반적인 고언어를 사용하는 것과 달리 고런타임이 얼마나 프로그래밍에서 다른지 명료하게
설명한다. 특정 인터페이스의 디테일보다 광범위한 관점에 집중에서 설명하려고
한다.

### Scheduler structures

스케쥴러는 런타임에 걸쳐서 세 가지 유형의 리소스로 운영한다: G, M, 그리고 P.
직접 스케쥴러를 통한 로직을 돌리지 않더라도, 이것을 이해하는 것은 중요하다.

#### Gs, Ms, Ps

"G" 는 간단하게 고루틴이다. type을 `g` 로 표현한다. 고루틴이 존재할 때, `g` 객체는
사용가능한 `g` 풀에 반환되고, 이것은 다른 고루틴에 의해 재사용될 수 있다.

"M" 은 OS Thread 인데, 유저의 Go code, 런타인 코드, 시스템 콜을 실행할 수 있으며,
idle 상태로 있을 수 있다. type을 `m` 으로 표현한다. 시스템 콜에서 쓰레드 수가 블록될 수도
있기 때문에 한 번에 임의의 수로 M이 있을 수 있다.

마지막으로, "P" 는 스케쥴러 및 메모리 할당하는 것과 같이 유저의 Go code를 설행하기 위해
요구되는 리소스를 나타낸다. type을 `p` 로 표현한다. GOMAXPROCS가 Ps와 동일하다.
P는 OS 스케쥴러에 있는 CPU와 같다고 생각할 수 있고, `p` type이 가지고 있는 내용은
각 CPU State 와 같다. 효율성을 위해 샤딩이 필요하지만, 쓰레드당 또는 고루틴일 필요없는
상태를 두기에 적절한 곳이다.

스케쥴러의 역할은 G(코드 실행), M(실행할 곳), 그리고 P(실행권한과 리소스)를 매칭하는 것이다.
Go code를 실행하는 것을 M이 멈출 때, 예를 들어 시스템콜이 들어온다고 가정하면, 이 P는 Idele P로
반환된다. 그리고 Go code를 재실행하기 위해선, 시스템 콜에서 반환되면, idle pool에서
P를 획득해야만 한다.

모든 g, m, 그리고 p 객체는 Heap에 할당되고, 절대 해제되지는 않는다.
따라서 안정적인 상태로 메모리에서 유지된다.
결론적으로 런타임은 스케쥴러 단에서 쓰기장벽(Write barriers)를 피할 수 있다.

#### User stacks and system stacks

모든 non-dead G는 Go code가 실행되는 User stack과 연결되어 있다. User stack은 2K와 같이
작은 값으로 시작하고, 동적으로 증가 및 축소된다.

모든 M은 연결된 시스템 스택(Stub G로 구현되기 때문에 M의 "g0" stack이라고 알려짐)이 있고,
유닉스 플랫폼에서의 Signal stack이 있다(M의 "gsignal" stack 이라고 알려짐).
System 및 Signal stack 모두 증가될 수 없지만, 런타임에 실행되고 cgo code를 실행하기에 충분히
크다(순수 Go binary에서는 8K; cgo binary에서 시스템할당됨).

런타임 코드는 systemstack, mcall, 또는 asmcgocall을 사용하여 종종 일시적으로 System stack으로
전환하여 선점되지 않아야만 하는 task를 수행하는데, 이는 User stack을 증가하지 말아야 하는 task나,
user goroutine을 전환하는 작업이 있다. System stack에서 실행되는 코드는 암묵적으로 비선점이고, 
Garbage collector는 System stack을 스캔하지 않는다. System stack이 실행되고 있는 동안,
현재의 user stack은 실행에 사용되지 않는다.

#### `getg()` and `getg().m.curg`

현재 유저의 `g` 를 획득하기 위해서 `getg().m.curg` 를 사용한다.

`getg()` 는 단독으로 현재의 `g` 를 반환하지만, system 또는 signal stack에서 실행될 때, 이는 현재의
m의 "g0" 또는 "gsignal" 을 각각 반환한다. 이것은 일반적으로 원하는 방식은 아니다.

User stack 또는 System stack에서 실행되고 있는지 확인하고 싶다면
`getg()` == `getg().m.curg` 인지 확인한다.

### Error handling and reporting

user code 로 합리적인 방식으로 recover 될 수 있는 에러는 일반적으로 panic을 사용해야 한다.
하지만, mallocgc 동안 system stack에서 호출되는경우와 같이 panic이 즉시 fatal error를 일으키는
경우도 있다.

대부분의 런타임 에러는 recover가 가능하지 않다. 이런 경우에는 throw를 사용한다. throw는 
traceback을 덤프하고 즉시 process를 종료한다. 일반적으로 throw는 위험한 상황에서
할당하는 것을 피하기 위해 string constant를 전달해야한다. 관례적으로,
추가적인 디테일은 print 또는 println으로 출력되고 "runtime: " 이라는 prefix 메시지가
붙는다.

런타임 error 디버깅을 위해, `GOTRACEBACK=system` 또는 `GOTRACEBACK=crash` fmf
함께 실행하는 것이 유용하다.

### Synchronization

런타임은 여러 동기화 메커니즘이 있다. 의미론 적으로 다른 것이 있는데, 특히 Goroutine scheduler 또는
OS scheduler와 상호작용하는 여부에 따라 다르다.

가장 단순하게 생각해보면 mutex가 있다. lock과 unlock을 사용하여 조작한다. 이것은 짧은 기간동안
공유된 structure를 보호하는데 사용된다. mutex에서의 블로킹은 Go scheduler와 상호작용하지 않는다면, 직접적으로 M을 블록한다. 이는 런타임에서 가장 낮은 레벨에서 사용되어 안전하다는 것을 의미한다.
뿐만 아니라, reschedule되는 것으로 부터 G와 P와 연관된 어떠한 것도 안전하게 처리한다. `rwmutex` 와 비슷하다.

one-shot notification을 위해 `note` 를 사용한다. `note` 는 `notesleep` 과 `notewakeup` 과 같은
함수를 제공한다. 전통적인 UNIX에서의 sleep/wakeup 과 달리, note는 race-free 이다. 그래서
notesleep은 notewakeup이 일어나자마자 즉시 반환된다. note는 sleep하거나 wakeup와
절대 race하지 않는 `noteclear` 와 함께 리셋될 수 있다. Mutex와 같이 note에서 블로킹하고 있다면 M을 블록한다. 
하지만 note에서 sleep하는 다른 방식도 있다: notesleep은 G와 P와 연관된 어떤 것도 rescheduling을
예방하는 반면에, notesleepg는 System call을 블로킹하는 것처럼 작동하는데 다른 G를 재사용하는 P를 허용한다.
이것은 M을 소비하기 때문에 G를 직접적으로 블로킹 하는 것보다 덜 효율적이다.

Gooutine scheduler와 직접적으로 소통하기 위해, `gopark` 와 `goready` 를 사용한다.
gopark는 현재의 goroutine을 "waiting" 상태에 두고 스케쥴러의 run queue에서 제거한다.
그리고 다른 goroutine을 현재의 M/P에 할당한다. goready는 parked goroutine을
다시 "runnable" 상태로 돌리고, run queue에 추가한다.

요약하자면 다음과 같다.

<table>
<tr><th></th><th colspan="3">Blocks</th></tr>
<tr><th>Interface</th><th>G</th><th>M</th><th>P</th></tr>
<tr><td>(rw)mutex</td><td>Y</td><td>Y</td><td>Y</td></tr>
<tr><td>note</td><td>Y</td><td>Y</td><td>Y/N</td></tr>
<tr><td>park</td><td>Y</td><td>N</td><td>N</td></tr>
</table>

### Atomics

런타임 패키지에서는 `runtime/internal/atomic` 에 있는 것으로 런타임만을 위한 원자성 패키지를 사용한다.
이것은 `sync/atomic` 과 일치하지만 역사적인 이유로 다른 이름의 함수명을 갖고 있고 런타임에 필요한 몇가지
추가 함수들이 있다.

일반적으로 우리는 불필요한 atomic operation을 피하기 위해 노력하고, 런타임에서 atomic하게 사용하는 것을
힘들게 생각한다. 변수 접근이 동기화 메커니즘에 보호되는 경우에, 이미 보호된 접근은 일반적으로 atomic하지
않아도 된다. 몇가지 이유가 있다.

1. non-atomic 또는 atomic 접근을 사용하는 것은 코드가 자체적으로 문서화된다. 변수에 대한 atomic 접근은
이 변수가 어딘가에서 동시에 접근될지도 모른다는 것을 함축하고 있다.
2. Non-atomic 접근은 자동으로 race detection에 대해서 혀용한다. 런타임은 현재 race detector를
가지고 있지 않다. 하지만 미래에는 필요할 수 있다. atomic 접근은 race detector를 무력화하는 반면,
non-atomic 접근은 race detector가 추론할 수 있도록 한다.
3. Non-atomic 접근은 성능을 향상시킬 수 있다.

물론, 공유된 변수에 대한 어떠한 non-atomic 접근은 어떻게 보호되고 있는지 설명이 문서화되어야만 한다.

일반적으로 atomic과 non-atomic 접근이 섞여서 구현되는 패턴은 다음과 같다:

- lock으로 보호되는 업데이트인 경우의 변수를 읽는 경우. locked region에서, 읽기는 atomic할 필요가 없다.
하지만 쓰기는 atomic해야한다. locked region 밖에서는 읽기는 atomic해야한다.
- 읽기가 STW(Stop the world)가 일어나는 동안에만 일어나는 경우, STW 동안에는 쓰기가 일어나지 않는다. 이때는 stomic하지 않아도 된다.

즉, Go memory 모델 관점으로의 조언은 "Don't be \[too\] clever." 이다. 런타임에서 성능도 중요하지만
견고성이 더 중요하다.

### Unmanaged memory

일반적으로 런타임은 regular heap allocation을 사용한다. 하지만 경우에 따라 런타임은 관리되지 않는 메모리에서
GC된 힙 외부의 객체를 할당해야한다. 객체가 memory manager 자체의 일부이거나
caller가 P가 없을 수도 있는 상황에서 할당되어야만 하는 경우에 필요하다.

Unmanaged memory가 할당되기 위해선 세 가지 메커니즘이 있다.

- sysAlloc은 OS에서 직접 메모리를 획득한다. 이것은 system의 page size의 배수로 제공되고
sysFree와 함께 해제될 수 있다.
- persistentalloc은 여러개의 작은 할당을 단일 sysAlloc으로 결합하여 fragmentation을 방지한다.
하지만 persistentalloced objects에 대해서 해제할 수 있는 방법은 없다.
(이름이 persistentalloced object인 까닭임)
- fixalloc은 SLAB-style allocator인데, 고정된 크기로 객체를 할당한다. fixallocaed object는
해제될 수 있지만, 이 메모리는 같은 fixalloc pool에 의해 재사용될 수 있다. 그래서 같은 type의 객체에 대해서만
재사용될 수 있다.

일반적으로, 이들 중 하나를 사용하여 할당된 type은 아래와 같이 `//go:notinheap` 으로 표시된다.

Unmanaged memory에 할당된 객체는 다음 규칙을 지키지 않는 한 heap pointer를 포함하지 않아야만 한다.

1. 힙으로의 Unmanaged memory로 부터 존재하는 pointer는 GC root이어야만 한다. 좀 더 구체적으로,
모든 포인터는 global variable을 통해 접근이 가능하거나 `runtime.markroot` 에서 명시적인 GC root로
추가되어야 한다.
2. 메모리가 재사용된다면, heap pointer는 GC root로 부터 표시되기 전에 zero-initialize되어야만 한다.
그렇지 않으면, GC는 stable heap pointer를 관찰할 수 있다.
"Zero-initialization versus zeroing"을 참고하길 바란다.

### Zero-initialization versus zeroing

type-safe state에서 이미 초기화 되었는지에 따라 런타임에서 zeroing하는 두가지 방법이 있다.

메모리가 type-safe state에 있지 않다면, 그건 "garbage"를 포함하고 있다는 것을 잠재적으로 의미하고 있다.
할당되고 첫사용을 위해 초기화중이기 때문에, 그리고 `memclrNoHeapPointers` 나 non-pointer 쓰기를
사용해서 zero-initialized 되어야만 한다. 이것은 write barrier를 수행하지 않는다.

이미 type-safe state에 있고 단순히 zero value로 세팅되어 있다면, 이는 regular writes,
`typedmemclr` , 또는 `memclrHasPointers` 를 사용해서 수행되어야 한다. 이는 write barrier를
수행한다.

### Runtime-only compiler directives

"go doc compile"에 문서화된 "// go:" directive 외에, 컴파일러는 런타임에서만 추가적인
directives를 지원한다.

#### go:systemstack

`go:systemstack` 은 함수가 system stack에서 실행되어야 함을 나타낸다. 이것은
special function prologue를 통해 동적으로 확인된다.

#### go:nowritebarrier

`go:nowritebarrier` 는 해당 함수가 어떠한 write barrier를 포함하고 있는지에 따라
컴파일러가 error를 내도록 한다.(이것은 write barrier의 생성을 억제하지 않음. 단순히 assertion임)

일반적으로, `go:nowritebarrierrec` 를 원한다. `go:nowritebarrier` 는 우선적으로 write barrier가
없는 것이 좋을 때 유용하지만, 바로잡는 것에는 사용되지 않는다.

#### go:nowritebarrierrec and go:yeswritebarrierrec

`go:nowritebarrierrec` 는 `go:yeswritebarrierrec` 까지 해당 함수 또는 반복적으로 호출하는 함수가
write barrier가 포함된 경우 컴파일러가 오류가 내도록 한다.

논리적으로, 컴파일러는 각 `go:nowritebarrierrec` 함수에서 시작되는 호출 그래프를 플러딩하고,
write barrier를 포함하는 함르를 만났을 때 error를 생성한다. 이 플러딩은
`go:yeswritebarrierrec` 함수에서 멈추게 된다.

`go:nowritebarrierrec` 는 무한 루프를 방지하기 위한 write barrier를 구현하는 데 사용된다.

이 두 directive는 스케쥴러에서 사용된다. write barrier는 활성 P(getg().m.p != nil)가 요구되며,
스케쥴러는 활성 P가 없이 종종 실행되기도 한다. 이런 경우에, `go:nowritebarrierrec` 는 P를 릴리즈하거나
P가 없이 실행되거나 하는 함수에 사용되고, `go:yeswritebarrierrec` 는 활성 P를 다시 획득할 때 사용된다.
이것들은 funtion-level annotation이기 때문에, P를 해제하거나 획득하는 코드를 두 함수로 분할해야할 수 있다.

#### go:notinheap

`go:notinheap` 은 type 선언에 적용된다. GC된 heap이나 stack으로 부터 할당되지 않아야함을 가리킨다.
특히, 이 type의 pointer는 `runtime.inheap` 체크에서 실패한다. 이 type은 global variable로
사용될 수도 있고, unmanaged memory에 있는 객체에 사용될 수 있다.
(e.g., `sysAlloc` , `persistentalloc` , `fixalloc` 또는 manually-managed span)

특히:

1. new(T), make([]T), append([]T, ...) 와 T의 implicit heaap 할당은 허용하지 않는다.
(implicit 할당은 런타임 때 허용되지 않음)
2. unsafe.Pointer 이외의 regular type 의 pointer는 기본 타입이 동일하더라도 `go:notinheap` type의 pointer로 변환될 수 없다.
3. `go:notinheap` 을 포함하고 있는 type은 그 자체로 `go:notinheap` 이다. struct와 array는
element가 있는 경우 `go:notinheap` 이다. map과 channel은 `go:notinheap` 이 허용되지 않는다.
이를 명시적으로 하기 위해, `go:notinheap` 을 함축하는 모든 type의 선언이 `go:notinheap` 으로 
표시되어야한다.
4. `go:notinheap` type의 pointer에서 write barrier는 생략될 수 있다.

마지막으로 `go:notinheap` 의 진정한 이점을 말하자면, 런타임은 low-level internal structure에서
memory barrier를 피하기 위해, 단순히 원치않거나 비효율적인 memory allocator를
피하기 위해 `go:notinheap` 을 사용한다. 이 메커니즘은 안정적이고, 런타임의 가독성을 손상시키지 않는다.

---

작성일: 2021-04-05

---

## 후기

Golang을 메인 언어로 사용한지 어느정도 시간이 흘렀다. 프로덕션에서 운영하며 다양한 use case를
만났는데, runtime panic을 만나는 경우도 있었고, 프로젝트의 소스코드 레벨에서 해석하기 어려운
상황도 발생했었다.

사내에서 Gopher들의 모임을 진행하며 고언어를 이해하는 방식이 달라졌는데, 이게 문제를 해결하는데
큰 도움을 줬다. 고언어의 구현체는 go code로 구현되어 있으며, 각 구현체의 디테일은
주석에 정리되어 있고 코드를 따라가면 동작원리를 이해할 수 있다는 점이었다.

언어를 공부할 때 언어를 구현한 코드를 보며 공부한다는 생각은 안했던 것 같은데, 고언어는
구현체를 직접 보고 이해하면서 습득하는 게 더 좋은 경험으로 동작했다.
이번에는 정리하지 않았지만 GC life cycle에서 어떤 동작을 하는지 찾아보다가, HACKING.md 파일에
이끌려 번역까지 하게 되었다.

다양한 공부방법이 존재하지만, 고언어를 이해한다고 한다면 가장 확실한 공부방법은
소스코드를 보고 구현체를 이해하는 것이라 생각한다. HACKING.md를 읽고 고언어 구현에 대해서
가볍게 시작하고 필요에 따라 각 요소에 따라서 코드를 직접보고 이해하는 것을 추천한다.
가장 단순한 언어 중에 하나이기 때문에 다들 어렵지 않게 이해할 것이라 생각한다.

## References

- [https://golang.org/src/runtime/HACKING.md](https://golang.org/src/runtime/HACKING.md)
- [https://github.com/golang/go](https://github.com/golang/go)