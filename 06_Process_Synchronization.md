# Process Synchronization

## 데이터 접근
- 여러 프로세스가 하나의 storage 값을 접근하는 경우 Race Condition의 가능성이 있음
- 일반적으로 프로세스는 자신의 데이터에만 접근 가능하기 때문에 문제가 생기지 않음
- 하지만 System call을 통한 운영체제 내의 데이터를 건드리는 과정에서 문제 발생 가능
  
## Race Condition
- kernel 수행 중 인터럽트 발생 시
  - 해결책: 해당 작업 수행 중 interrupt를 disable 시키도록
- Process가 system call을 하여 kernel mode로 수행 중에 context switch가 일어나는 경우
  - 해결책: kernel mode인 경우에는 CPU를 preempt하지 않음
    - kernel mode에서 user mode로 돌아갈 때 preempt
- Multiprocessor에서 shared memory 내의 kernel data
  - 해결책1: 한번에 하나의 CPU만 커널에 들어갈 수 있도록
  - 해결책2: 커널 내부에 있는 각 공유 데이터에 접근할 때 마다 그 데이터에 대한 lock/unlock
    - 해결책1 보다 적은 오버헤드

## Process Synchronization 문제
- 공유 데이터의 동시 접근(concurrent access)은 데이터의 불일치 문제(inconsistency)를 발생시킬 수 있다
- 일관성(consistency) 유지를 위해서는 협력 프로세스(cooperating process) 간의 실행순서(orderly execution)을 정해줄 필요
- Race Condition
  - 여러 프로세스들이 동시에 공유 데이터를 접근하는 상황
  - 데이터의 최종 연산 결과는 마지막에 그 데이터를 다룬 프로세스에 따라 달라짐
  - 이를 해결하기 위해 concurrent process는 동기화(synchronize)되어야 한다

## The Critical-Section Problem
- n개의 프로세스가 공유 데이터를 동시에 사용하기를 원하는 경우
- 각 프로세스의 code segment에는 ***공유 데이터를 접근하는 코드인 critical section***이 존재
- 하나의 프로세스가 critical section에 있을 때 다른 모든 프로세스는 critical section에 들어갈 수 없어야 한다
- 해결을 위해(동기화를 위해) 몇몇 변수들을 프로세스들이 공유
  - synchronization variable

- 해결책1
  - 반드시 한번씩 교대로 들어가야만 한다(swap-turn)
  - turn이라는 synchronization variable 사용
  - 특정 프로세스가 더 빈번히 critical section을 사용해야 한다면?
    - 다른 프로세스가 critical section을 사용할 때 까지 기다려야만 한다
    - Mutual Exclusion 충족, Progress의 문제
- 해결책2
  - 프로세스 별 flag 사용 하여 critical section 접근 의사 표시
  - 다른 프로세스의 flag 확인, true인 경우 기다린다
  - 사용 후 flag = false
  - Mutual Exclusion 충족, Progress의 문제
    - 두 프로세스가 flag = true이지만, 작업중이 아닌 경우 무한히 대기할 수 있음
- 해결책3(Peterson's Algorithm)
  - flag와 turn을 모두 사용
  - 모든 조건 충족
  - flag를 들고 turn을 상대의 차례로 바꿈
  - flag를 우선 사용, 이후 turn 사용
    - flag를 여럿이 든 경우 turn으로 progress 문제 해결
  - busy waiting(=spin lock) 문제 발생
    - cpu를 가지고 대기하는 동안 while문을 지속적으로 계산하게 된다
- 해결책4(Synchronization Hardware)
  - 하드웨어적으로 Test & Modify를 atomic하게 수행할 수 있도록 지원하는 경우 해결됨
  - `Test_and_Set(a)` a를 읽고 바꾸는 작업을 atomic 하게 한번에 할 수 있는 명령어
- 해결책5(Semaphores)
  - 앞의 방식들을 추상화시킴
  - Semaphore S (일종의 자원 개수)
    - integer value
    - 아래의 두가지 atomic 연산에 의해서만 접근 가능
    - P(S): if positive->decrement&enter, otherwise->wait until possible(=busy wait)
      - lock을 걸고 자원을 획득
    - V(S): S++
      - lock을 해제
    - Mutual Exclusion 문제에서는 S가 1인 경우와 마찬가지
- Block & Wake up Implementation(busy waiting 해결책)
  - Semaphore S를 먼저 decrement, 이후 S가 음수인 경우 CPU를 반납, blocked 상태로 돌린다
  - block: 커널은 block을 호출한 프로세스를 suspend시킴, 이 프로세스의 PCB을 semaphore에 대한 wait queue에 넣음
  - wake up(P): block된 프로세스 P를 wake up 시킴, 이 프로세스의 PCB를 ready queue로 옮김
  - sleep lock 상태
- 일반적으로 Block & wake up 방식이 더 좋음
  - 오버헤드가 있기 때문에, critical section의 길이가 매우 짧은 경우(==경쟁이 적은 경우) busy wait 오버헤드보다 더 커질 수 있음
- 해결책6(Monitor)
- 해결책의 충족조건
  - Mutual Exclusion
    - 프로세스간 상호 배제
  - Progress
    - 아무도 Critical section에 있지 않은 상태에서 critical section에 들어가고자 하는 프로세스가 있다면 들어가야
  - Bounded Waiting(유한대기)
    - starvation이 일어나지 않아야 한다
