# JVM 의 구조는 어떻게 되어 있을까

Java 라는 프로그래밍 언어가 등장하고 나서 Java 를 이해하기 위해 JVM 의 구조에 대해 설명하는 많은 글이 있다.   
다른 포스팅된 글을 참고하여 나만의 것으로 만들기 위한 내용들을 작성한다.

## JVM 이란?

JVM 은 JRE(Java Runtime Environment) 의 구성 요소 중 하나로 Java Virtual Machine 의 약자이며 Java 바이트코드를 해석하고 실행한다.   
JVM 상에서 작동하는 언어들은 Compile 시 Java 바이트코드로 결과물이 나온다.
 
### JVM 상에서 작동하는 언어들

- Java
- Kotlin
- Groovy
- Scala
- Clojure
- ...
위 언어들 뿐만 아니라 기존 언어도 JVM 상에서 작동 될 수 있도록 Java 바이트코드로 Compile 하는 Jython, JRudy 등 또한 존재한다.

### Feature.

- Stack 기반의 가상 머신: x86 아키텍쳐나 ARM 아키텍쳐와 같은 대표적인 하드웨어 컴퓨터 아키텍쳐가 레지스터 기반으로 동작하지만 JVM 은 Stack 기반으로 동작한다.
- Symbolic Reference: Primitiva Data Type 을 제외한 모든 타입을 Symbolic Reference 를 통해 참조한다.
- Garbage Collection(GC): Class 인스턴스는 개발자에 의해 명시적으로 생성되고 GC 에 의해 자동으로 삭제된다.
- Network Byte Order: x86 아키텍쳐가 사용하는 Little Endian 과 RISC 계열 아키텍쳐가 주로 사용하는 Big Endian 사이에 플랫폼 독립성을 유지하기 위해 Java Class 파일은 네트워크 전송 시에 사용하는 Byte Order 인 Network Byte Order 를 사용한다.

## Java Byte Code

- WORA(Write Once Run Anywhere) 를 위해 JVM 은 Human Readable 언어인 Java 와 기계어 사이인 Java Byte Code 를 사용한다.
- Java Code 를 배포하는 가장 작은 단위이다.

## JVM 구조
![JVM Code Execition Process](img/JVM_code_execution.png)

### Reference.

- https://d2.naver.com/helloworld/1230
- https://catsbi.oopy.io/df0df290-9188-45c1-b056-b8fe032d88ca
