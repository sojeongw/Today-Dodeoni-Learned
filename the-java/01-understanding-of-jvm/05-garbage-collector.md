# Garbage Collector

Garbage Collection을 수행하는 부분

## Garbage Collection

동적으로 할당된 메모리 중에서 시스템에서 더 이상 사용되지 않는 블럭을 찾아 사용 가능하도록 회수하는 방법이다. 

C언어의 경우 프로그래머가 메모리를 할당하고 해제까지 수동으로 직접 해줘야 한다. 그래서 실수로 해제하지 않아 메모리 누수가 발생하거나 버그가 발생한다.

이런 문제를 해결하기 위해 자바에서는 Garbage Collection을 채택하고 있다. 메모리의 할당과 해제를 자동으로 실행하며 메모리를 관리해준다.

## Garbage Collector가 하는 일

- 메모리 할당
- 사용 중인 메모리 인식
- 사용하지 않는 메모리 인식

즉, 메모리가 부족할 때 쓰레기를 정리해주는 프로그램이다.

## Garbage Collector의 원리

Java는 JVM이 메모리를 부여받고 프로그램을 실행하다가 메모리가 부족해지면 추가 메모리를 요청한다. 이때 Garbage Collector가 실행된다. 순서는 다음과 같다.

1. 메모리를 관리하는 OS에 프로그램 실행에 필요한 만큼을 요청한다.
2. 메모리를 저장할 주소인 offset 주소를 할당한다.
3. 프로그램이 돌아가면서 할당된 메모리에 garbage가 발생한다.
    - 기존에 가리키던 메모리를 새롭게 선언하거나 형변환을 하면서 주소를 잃어버리면 다시 찾을 수 없게 되어 정리되지 않은 메모리가 발생한다.
4. Garbage Collector가 해당 garbage를 다른 용도로 사용할 수 있도록 메모리를 해제시킨다.

## Stop-The-World

GC 실행을 위해 JVM이 애플리케이션 실행을 멈추는 것. 

Stop-The-World가 발생하면 GC를 실행하는 스레드를 제외한 모든 스레드가 작업을 멈춘다. 그래서 대부분이 말하는 GC 튜닝은 Stop-The-World의 시간을 줄이는 것을 의미한다.

## Mark and Sweep

GC의 과정을 일컫는 말.

- Mark
    - Garbage Collector가 닿을 수 있는 변수나 객체를 스캔하면서 어떤 객체를 가리키고 있는지 찾는 과정
    - 이 과정에서 Stop-The-World가 발생함
- Sweep
    - Mark 되어있지 않은 객체를 heap에서 제거하는 과정
    
## Garbage Collector의 한계

- 어떤 방식의 쓰레기 수집을 사용하든 실행 시간에 작업을 수행하는 이상 성능 하락을 피할 수 없음
- Garbage Collector가 존재하더라도 더 이상 접근 불가능한 객체만 회수하므로 메모리 누수는 발생할 수 있음