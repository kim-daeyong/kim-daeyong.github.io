---
layout: post 
title: JVM heap profiling
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, JVM, Heap]
categories : [Programming]
toc: true
---


### TRY  
자바 앱의 메모리의 사용량이 높게 나와 메모리 누수를 확인하려고 했다.  
GC등 정상적으로 돌고있는 것 같은데 메모리가 조금씩 올라가는 것 같았다.  
그래서 heap dump 분석하려한다.  

### CATCH  

* 자바앱의 PID 확인  
```shell

$ jps

1234 Jps
5678 jvm_app_name

```  

*  heap 확인  
```shell
$ jmap -heap {pid}

Attaching to process ID 5678, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 17.0.7+7-LTS

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 759169024 (724.0MB)
   NewSize                  = 225771520 (215.3125MB)
   MaxNewSize               = 253034496 (241.3125MB)
   OldSize                  = 451608576 (430.6875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 22020096 (21.0MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 203292672 (193.875MB)
   used     = 120264008 (114.69269561767578MB)
   free     = 83028664 (79.18230438232422MB)
   59.15806350363677% used
Eden Space:
   capacity = 180748288 (172.375MB)
   used     = 111657200 (106.48460388183594MB)
   free     = 69091088 (65.89039611816406MB)
   61.77496961962926% used
From Space:
   capacity = 22544384 (21.5MB)
   used     = 8606808 (8.208091735839844MB)
   free     = 13937576 (13.291908264160156MB)
   38.17717086437137% used
To Space:
   capacity = 22544384 (21.5MB)
   used     = 0 (0.0MB)
   free     = 22544384 (21.5MB)
   0.0% used
tenured generation:
   capacity = 451608576 (430.6875MB)
   used     = 71497664 (68.18548583984375MB)
   free     = 380110912 (362.50201416015625MB)
   15.831777295566681% used
```  

이외에 통계도 볼 수 있다.  
```shell
$ jmap -histo:live {pid}

 num     #instances         #bytes  class name (module)
-------------------------------------------------------
   1:        109104        7053472  [B (java.base@17.0.7)
   2:          9595        2988968  [I (java.base@17.0.7)
   3:         73803        2361696  java.util.concurrent.ConcurrentHashMap$Node (java.base@17.0.7)
   4:         98147        2355528  java.lang.String (java.base@17.0.7)
   5:         15770        1864480  java.lang.Class (java.base@17.0.7)
   6:         18148        1597024  java.lang.reflect.Method (java.base@17.0.7)
   7:         29712        1188480  java.util.LinkedHashMap$Entry (java.base@17.0.7)
   8:         16675         931768  [Ljava.lang.Object; (java.base@17.0.7)
   9:          8708         745200  [Ljava.util.HashMap$Node; (java.base@17.0.7)
  10:           888         658048  [Ljava.util.concurrent.ConcurrentHashMap$Node; (java.base@17.0.7)
  ..

```  

이런 에러가 나올 수 있다.  
```shell

Error: -heap option used
Cannot connect to core dump or remote debug server. Use jhsdb jmap instead
```  

이땐 그냥 jhsdb를 사용하면 된다.  

```shell
$ jhsdb jmap --heap --pid 5678
```  

* heap dump file 생성  
```shell
$ jmap -dump:format=b,file=heapdump.hprof {pid}

$ jcmd {pid} GC.heap_dump ./heapdump.hprof

# jhsdb의 경우 jvm이 정지되니 주의하자.
# 서버 에러가 나거나 운영중이 아닌 곳에서만 사용
$ jhsdb jmap --binaryheap --dumpfile heapdump.hprof --pid {pid}

혹은 실행 시 
$ ‐XX:+HeapDumpOnOutOfMemoryError 추가

```  

* 각 지표는 뭘까  

우리 gpt 선생님의 말씀을 들어보자  
```  
Shallow Objects (얕은 객체): Shallow 객체는 개별 객체의 직접 메모리 크기를 나타냅니다. 이것은 해당 객체 자체의 크기이며 하위 객체(참조하는 다른 객체)의 크기는 포함하지 않습니다.

Retained Objects (보유 객체): Retained 객체는 특정 지배자(dominator) 객체와 그 하위 객체의 메모리 크기의 합계를 나타냅니다. 이것은 메모리 누수 분석에 중요한 지표로 사용됩니다. 메모리에서 삭제해도 다른 객체에 의해 참조되는 객체들을 식별하는 데 도움이 됩니다.

Biggest Objects (가장 큰 객체): Biggest Objects는 HProf 파일에서 가장 큰 메모리를 사용하는 객체들을 나타냅니다. 이것은 개별 객체의 크기를 기준으로 정렬되며, 애플리케이션 내에서 가장 큰 메모리 사용을 하는 객체들을 확인하는 데 사용됩니다.

GC Root (가비지 컬렉션 루트): GC Root 객체는 메모리에서 시작하는 객체 참조 체인의 루트 역할을 하는 객체를 나타냅니다. 가비지 컬렉션 동안 삭제되지 않는 객체들은 GC Root에서부터 참조되고 있음을 나타냅니다.

MergedPath (병합된 경로): MergedPath는 객체가 GC Root까지의 경로를 나타냅니다. 이것은 객체가 GC Root에 도달하기 위해 어떤 경로를 통과하는지를 설명하는 데 사용됩니다.

Packages (패키지): Packages 정보는 HProf 파일에서 사용된 클래스의 패키지 정보를 나타냅니다. 패키지별로 클래스 수와 메모리 사용량을 보여주어 어떤 패키지가 메모리 사용에 영향을 미치는지를 확인하는 데 도움이 됩니다.
```  

힙 덤프 에서 봐야하는 건?

```
Total Heap Size (전체 힙 크기): HProf 파일에는 전체 힙 크기에 대한 정보가 포함되며, JVM에서 할당한 총 힙 메모리 크기를 나타냅니다. 이 값은 JVM 실행 시 -Xmx 옵션으로 설정한 값에 해당합니다.

Used Heap (사용 중인 힙): 사용 중인 힙은 현재 할당된 객체에 의해 사용 중인 힙 메모리 크기를 나타냅니다. 이 값은 HProf 파일이 생성된 시점에서의 실제 메모리 사용량을 보여줍니다.

Free Heap (사용 가능한 힙): 사용 가능한 힙은 힙 내에서 아직 할당되지 않은 메모리 크기를 나타냅니다. 이것은 전체 힙 크기에서 사용 중인 힙을 뺀 값과 동일합니다.

Live Objects (살아 있는 객체 수): HProf 파일에는 현재 메모리 힙 내에 있는 활성 객체(참조되고 있는 객체)의 수가 포함됩니다.

Garbage Objects (가비지 객체 수): 가비지 객체는 더 이상 참조되지 않는 객체를 나타냅니다. HProf 파일은 이러한 가비지 객체의 수를 기록하며, 가비지 컬렉션(Garbage Collection)에 의해 회수될 것입니다.

Dominators (지배자): 지배자는 다른 객체에 대한 루트 역할을 하는 객체를 나타냅니다. 이러한 객체는 메모리 누수(Memory Leak)와 관련된 원인을 파악하는 데 도움이 됩니다.

Retained Size (보유 크기): 보유 크기는 지배자 객체와 해당 하위 객체의 메모리 크기의 합계를 나타냅니다. 이 값은 메모리 누수 원인을 찾는 데 중요한 지표입니다.

Reference Chains (참조 체인): 특정 객체가 다른 객체에 어떻게 참조되고 있는지를 보여주는 참조 체인 정보를 분석하여 메모리 누수 및 순환 참조를 식별하는 데 도움이 됩니다.
```

* 메모리 leak 상황 참고. 

[java memoru leak-baeldung](https://www.baeldung.com/java-memory-leaks)


### FINALLY  
이같이 heap을 보고 코드를 수정하거나, 혹은 GC 튜닝을 통해서 해결할 수 있을 것 같다.  

끝

---
