Question)

1.
순차적이라는 것은 병렬처리와 거리가 있어보이는데 그렇다면 스트림은 이러한 순차적인 처리에서 어떠한 방식으로 병렬처리를 진행하는가?(parallelStream()에서만 병렬처리가 가능한가?)

 A. 스트림의 각 요소를 파이프라인에 정의된 연산으로 순차적으로 처리한다.
 
 스트림
 [1,2,3,4,5,6]
 
 필터(짝수만 골라) - 1탈락 2반환 3탈락 4반환 5탈락 6반환
 map(제곱하라)     - x     4반환  x    16반환 x    36반환
 forEach           - x     4      x    16     x    36
 
 1,2,3,4,5,6의 요소를 여러 쓰레드에 할당한 후 파이프라인 연산을 실행한다.
 해당 부분은 fork&join 프레임워크에서 상세히 나온다.
 ```java
 //Please don't change class name 'Main'
import java.util.*;


class Main {

  public static void main(String[] args) {
  
   //Optional<String> a =   
         Arrays.asList("a", "b", "c", "d", "e","f", "g", "h", "i", "z","k", "l", "m", "n", "o", "p")
      .parallelStream()
         //.stream()
          .map(s -> {
            System.out.println(String.format("Current Thread : %s, map : %s", Thread.currentThread().getName(), s));
            return s;
       })
      .filter(s -> {
            System.out.println(String.format("Current Thread : %s, filter : %s", Thread.currentThread().getName(), s));
            return true;
       })
        .sorted((s1, s2) -> {
            // System.out.printf("sort : %s, %s\n", s1, s2);
            System.out.println(String.format("Current Thread : %s, sorted : %s, %s", Thread.currentThread().getName(), s1, s2));
            return s1.compareTo(s2);
         })//a2,a3,a4
            // .limit(2)
            //       .map(s -> {
            // System.out.println(String.format("Current Thread : %s, limit after map : %s", Thread.currentThread().getName(), s));
            // return s;
            // })
       .forEach(s -> {
            System.out.println(String.format("Current Thread : %s, forEach : %s", Thread.currentThread().getName(), s));
       }); //옵셔널제거하고 실행
         //.findFirst(); //옵셔널로 실행
      
      //System.out.println(a);

  }
}
 ```

2.
findFirst(), findAny() 는 어떻게 구별하여 사용하는가?
병렬처리에서 첫 번째 요소를 찾기 힘들다. 왜냐면 병렬적으로 처리하기 때문에 어떠한 것이 첫 번쨰 요소인지 구별하기가 힘들기때문이다.
이로인해 병렬처리에서는 findAny() 메서드를 사용한다.
-> 책에서는 이렇게 설명되어있다.

​ 그렇다면 findFirst()를 병렬처리에서 구현한다면 구체적으로 어떠한 에러를 뱉어내는가? 첫 번째 요소인지를 구별하기 위한 방법은 정말 없는가?

A. 책에서 나오는 찾기 힘들다의 의미는 찾지 못한다의 의미가 아니라 찾는데 시간이 오래걸린다는 의미로 받아들여야된다. 병렬로 데이터 건수가 많으면 병렬로 findAny를 사용하면 적합할지 모르지만 findFirst를 사용하면, 다중의 스레드에서 찾은 해당 요소가 스트림의 첫번째 조건에 맞는 요소인지에 대해서는 구분하기 어렵다.


3.
 iterate와 generate를 통해 각각 피보나치 수열을 구현했을 떄 차이점.
iterate는 람다식을 이용해 상태값이 변하지 못하지만,  generate를 통해 구현을 하게되면 람다식이 아닌 아래처럼 익명 클래스를 통해 구현을 해야한다.
이는 generate의 파라미터가 supplier로 되어있기 때문이다.
익명 클래스를 통해 구현을 하게 된다면 가변 상태가된다. 인스턴스 변수에 어떤 요소가 있었는지 추적하므로 가변 상태의 객체로 본다.
```java
IntSupplier fib = new IntSupplier() {
	private int previous = 0; //해당 변수가 가변상태가 된다.
	private int current = 1; //해당 변수가 가변상태가 된다.
			
	@Override
	public int getAsInt() {
		int oldPrevious = this.previous;
		int nextValue = this.previous + this.current;
		this.previous = this.current;
		this.current = nextValue;
		return oldPrevious;
	}
};
```

4.
왜 병렬 코드에서는 공급자에 상태가 있으면 안전하지 않다는 것인지 구현해서 알려주십시오.
'''java
//Please don't change class name 'Main'
import java.util.*;
import java.util.stream.IntStream;

public class Main {

  public static void main(String[] args) {
  for (int i = 0; i < 5; i++) {

            Set<Integer> seen = new HashSet<>();
            IntStream stream = IntStream.of(1, 2, 1, 2, 3, 4, 4, 5,1,2,3,4,4,5,5);
       
     int sum = stream.parallel()
		.map(
             
                e -> {
			System.out.println(seen.toString());          
			System.out.println( (String.format("%s, 숫자넣는다 %s", Thread.currentThread().getName(), e)));		
			System.out.println("### end ####");
                      if (seen.add(e))
                          return e;
                      else
                          return 0;
                  })
		.sum();

            System.out.println("::::::::::::::::::::::::::::::::::::::::::::::::"+sum);
        }
  }
  
}

'''

6.
page 142 : 파이프라인에서 데모, 디버깅 기법과 마찬가지로 출력 코드를 추가하지 않는 것이 좋다고 했는데 이유가 궁금합니다.
프로젝트에서 sysout을 지양하고 log4j를 사용하도록 권장하는 이유와 같다.
파이프라인에서 각 연산마다 출력 코드를 작성해두면 해당 출력 코드가 처리되기전까지 다른 행동을 하지않는다(해당 자원에 대해 sync처리가 되고있다.). 따라서 하나의 웹에 다수의 인원이 접속할 경우 한명이 잡고 있는 출력 코드로 인하여 다른 인원의 코드까지 락이 걸릴수 있다.


7.
page 167 : 실전연습 5.5 하나씩 설명 부탁드립니다.

그림으로 설명하겠음.


8.
p.144 builder pattern(빌더패턴)과 스트림의 연산은 유사하다고 합니다. 빌더패턴이 무엇인지, 개념 설명과 함께 간단한 예제를 들어주면 좋겠습니다.
빌더 패턴이란 객체를 생성할 때 빌더를 이용해서 생성하는 방법으로 아래와 같은 코드입니다.
```java
Member customer = Member.build()
    .name("홍길동")
    .age(30)
    .build();
```

9.
p.148 distinct 메서드를 통해 중복된 값을 제거할 수 있다고 합니다. 예제에서는 정수를 예로 들어 직관적으로 와닿지만, 실제 업무에서는 객체를 비교하는 일이 많습니다. distinct를 객체에 적용하여 중복을 제거하려면 어떻게 해야할까요?(교재를 예로 들면 Apple 객체 간의 중복된 데이터를 제거하는 방법)

객체에서 distinct를 사용해서 중복값을 제거하기위해서는 equals(),  hashCode()가 재정의되어야 합니다. 또는 predicate를 재정의해서 filter에서 거르는 방법을 써야합니다. https://stackoverflow.com/questions/23699371/java-8-distinct-by-property

10.
p.164 구글의 웹 검색은 맵-리듀스를 이용한다고 하는데, 조금 더 자세히 동작하는 방법에 대해 설명을 듣고 싶습니다.(같이 이야기 해보면 좋을 것 같은 주제)
https://12bme.tistory.com/154
