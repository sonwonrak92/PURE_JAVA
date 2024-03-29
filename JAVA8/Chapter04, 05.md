

# 스트림과 컬렉션

## Stream이란 ? 

- **컬렉션 데이터**, **배열 데이터**를 하나씩 참조하며 **함수형 인터페이스를 적용하고 반복적**으로 처리할 수 있도록 해주는 기능.
- 쉽게 말해서 컬렉션타입의 데이터들을 SQL질의어처럼 가공할 수 있게 해주는것이라 생각하면 됩니다.
- 스트림의 각 요소를 선언된 스트림메소드에 따라 처리하겠다. 라는 의미!!
- 따라서, 스트림화를 하게되면 파이프라인을 만들어서 데이터의  흐름을 손쉽게 파악할 수 있다.

> 컬렉션이란?
>
> [자바공식문서]: https://docs.oracle.com/javase/tutorial/collections/implementations/index.html



## Stream의 구조

- "컬렉션같은 객체집합.*스트림생성()*.*중간연산()*.*최종연산()*;"
- 서로 **연결가능한 스트림 연산을 중간연산**이라고하며, **스트림을 닫는 연산을 최종 연산**이라고 한다.

```sql

SELECT LENGTH, NAME, AGE --최종연산
FROM COLLECTION --컬렉션 스트림
WHERE LENGTH > 100 --중간연산
GROUP BY LENGTH, NAME, AGE
ORDER BY AGE

SQL언어와 비교해보자면 이런형식이 아닐까 생각합니다.
```

### #중간연산

- 중간 연산은 다른 스트림을 반환한다. 또한 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한번에 처리한다. 하지만 **중간 연산으로는 어떠한 결과도 생성할 수 없다.**

- 중간 연산은 LAZY하다

  ```JAVA
  Stream<String> a = names.stream()
      					.filter(x -> x.contains("o"))
      					.map(x-> x.concat("s")); 
  //중간연산만 존재하므로, 지금은 연산을 실행하지 않는다.
  
  a.forEach(x -> System.out.println(x)); //해당 시점에서 실행한다.
  
  //해당 코드에서 filter와 map은 forEach와 같은 최종 연산이 적용될 때 실행된다.
  ```

- 중간연산의 종류

  - filter, distinct, skip, limit,map 등

### #최종연산

- 최종 연산은 스트림 파이프라인에서 결과를 도출한다. **스트림 이외의 결과가 반환**된다.
- 최종연산의 종류
  - anyMatch, noneMatch, allMatch, forEach, Colect 등



## Stream 적용이 가능한 것들

- I/O resource(ex.file), Generators, Stream ranges, Pattern 등이 있다.

  -Stream생성의 대표적인 방법

```java
//Collection에서 스트림 생성
List<String> names = Arrays.asList("Lee", "Im", "choi", "son", "chae");
names.stream(); 
 
//배열로 스트림 생성
Double[] dArray = {3.1, 3.2, 3.3};
Arrays.stream(dArray);

// 스트림 직접 생성
Stream<Integer> str = Stream.of(19,20); 


출처: https://jeong-pro.tistory.com/165 [기본기를 쌓는 정아마추어 코딩블로그]
```



## CORE JAVA(명령형)와 스트림(선언형) 비교

```JAVA

[메뉴판에서 칼로리가 400 이하인 메뉴만 담아서 칼로리 순으로 정렬해달라.]

/**CORE JAVA - 명령형(절차형)으로 구현**/
//1 - 필터링 로직구현
for(Dish d : menu){
    if(d.getCalories() < 400){
        lowCal.add(d)
    }
}
//2 - 정렬 로직구현
Collections.sort(lowCal, new Comparator<Dish>(){
	public int compare(Dish d1, Dish d2){
		return Integer.compare(d1.getClaories(), d2.getCalories());
	}
})
//3 - 요리명추출 로직 구현
for(Dish d : lowCal){
 loCalName.add(d.getName());
}
//방법(어떻게 how)을 기술
1.메뉴의 각 요소들을 d에 담고 반복문을 돌리는데 만약 칼로리가 400 이하면 lowCal에 담는다.(필터링)
2.담아둔 lowCal의 각 요소를 반복문을 통해서 칼로리순으로 정렬한다.(정렬)
3.정렬한 lowCal의 메뉴명을 반복문을 통해서 lowCalName 배열에 담는다.(요리명추출)
    
    
/**STREAM - 선언형(함수형)으로 구현**/  
List<String> lowCalName =   menu.stream()
    							.filter(d -> d.getCalories() < 400) //1 - 필터링 선언
    							.sorted(comparing(Dish::getCalories)) //2 - 정렬 선언
    							.map(Dish::getName) //3 요리명추출 선언
    							.collect(toList()); 
//목표(무엇을 what)만 기술
1.400칼로리 이하만 추출
2.칼로리순으로 정렬
3.요리명추출
```



------



# 외부반복과 내부반복

- for문을 사용하여 반복문을 작성하는것을 외부반복이라고 보면된다.
- 반대로 stream 내부의 filter, map 등에 포함되어있는 반복문을 내부반복이라고 한다.

[외부반복과 내부반복의 성능차이]: https://codeday.me/ko/qa/20190618/821385.html



# for-loop와 Stream forEach의 사용의 차이

```java
for (int i = 0; i < 100; i++) {
  //Do something
  if (i > 50) {
    break;
  }
  System.out.println(i);
}
//for문의 경우 50번의 행위를 한 후 더이상 반복문을 수행하지않음.

IntStream.range(1, 100).forEach(i -> {
  //Do Something
  if (i > 50) {
    return;
  }
  System.out.println(i);
});
//stream의 forEach의 경우 중단할 수 없기에 100번의 행위를 모두 실행한다. 따라서 아래와 같이 필터로 해당 범위를 줄여주는 방법을 사용할 수 있다.

IntStream.range(1, 100)
         .filter(i -> i <= 50)
         .forEach(System.out::println);
```



# 일반 스트림과 병렬 스트림의 차이

> **stream()을 호출할지 parallelStream()을 호출할지 결정할 때 몇 가지 이슈를 고려해야 한다.** 
> 첫째, 람다 표현식을 동시에 실행하고 싶은가? 
> 둘째, 코드는 사이드 이펙트나 레이스 컨디션[^1]없이 독립적으로 수행 가능한가? 
> 셋째, parallelStream()이 제대로된 선택이려면 동시에 실행되도록 스케줄된 람다 표현식의 실행 순서에 독립적이어야 한다.
>
>  예를들어 forEach() 메소드의 호출과 람다를 통한 결과 출력을 병렬화 하는것은 옳지 않다.
> 병렬화된 코드에서는 어떤 메소드가 먼저 실행되고, 어떤 메소드가 나중에 실행되는지 순서를 예측할 수 없기 때문에 출력 순서도 뒤바뀔 수 있다.
>
>  다만, map() , filter() 같은 메소드는 병렬화하기 적합하다.

- 병렬 스트림은 내부적으로 **fork&join 프레임워크**를 이용해서 자동적으로 연산을 병렬로 수행한다

```java
//Please don't change class name 'Main'
import java.util.Arrays;
import java.util.List;
import java.util.Comparator;
import java.util.stream.Collectors;

import static java.util.stream.Collectors.toList;
class Main {

public static void main(String[] args) { 
  // TODO Auto-generated method stub 
  List<Integer> numbers = Arrays.asList(1,18 ,2, 8, 9, 10, 11, 12, 13, 14,3, 4, 5, 6, 7); 

  List<Integer> twoEvenSquares = 
   //numbers.stream() 
  numbers.parallelStream() 
         .filter(n -> {
        System.out.println("Current Thread filter : " + Thread.currentThread().getId()); 
        System.out.println("filtering " + n);                
               return n % 2 == 0; 
          }) 
          .map(n -> { 
         System.out.println("Current Thread mapping: " + Thread.currentThread().getId()); 
         System.out.println("mapping " + n);              
             return n; 
         })
        .sorted(Comparator.comparing(numbers::get) )
         //.limit(2)       
         .collect(toList()); 
      
   System.out.println(twoEvenSquares.size());
   } 

}
```



# CORE JAVA 와 STREAM 사용에 대한 고찰

## #가독성 관점에서

- **stream API의 숙달상태에 따라서 가독성에 대한 판단여부**가 달라진다.

회사에서 휴가를 신청하는 로직에 대해서 설명해보자.

먼저 회사의 휴가신청 프로세스를 잘 숙지하고 있는 경력사원과 숙지가 덜 된 신입사원의 입장에서 가독성에 대한 판단을 물어보면 아래와 같다.

```
##명령형으로 인한 가독성 판단
1.먼저 크롬을 켜서 VPN을 접속해야하는데 VPN접속방법은 CISCO설치 후 .... [VPN접속]
2.VPN을 접속했으면, 그룹웨어로 들어가는데 주소는 ... 들어가서 휴가신청서를 누릅니다.[휴가신청서]
3.신청서를 누른 후 팝업창이 뜨면 본인의 휴가가능일자를 찾아서 선택합니다.
4.선택된 휴가에 대해서 사유를 입력한 후 결재자 찾기를 결재자 찾기는 팝업 상단에 위치하며 ...
  팝업이 뜨면 눌러서 팀장님으로 지정을 합니다. [결재자 지정]
5.선택된 휴가에 대해 이상이 없으면 결재버튼을 누른 후 결재를 기다립니다.[결재신청]

-신입사원 : 오.. 상세하게 잘나와있어서 따라하기 쉽네
-경력사원 : 귀찮게 말 길게하네.

##선언형으로 인한 가독성 판단
1.VPN접속합니다.[VPN접속]
2.그룹웨어에서 휴가신청서버튼 누릅니다.[휴가신청서]
3.휴가가능일자 선택한 후 결재자 지정합니다.[결재자지정]
4.이상이 없으면 결재버튼을 누른 후 기다립니다.[결재신청]

-신입사원 : VPN은 뭐야? 어떻게 깔아? 그룹웨어는 어떻게 들어가?
-경력사원 : 간단하네.
```



## #성능 관점에서

[for-loop VS Stream 성능비교]: https://jaxenter.com/java-performance-tutorial-how-fast-are-the-java-8-streams-118830.html

결론 : 속도차이의 경우는 로직의 상태에 따라 달라진다. 

​           신기술이라고 해도 무조건적으로 stream이 빠르다고 할 수 없다.



# 스트림 활용

## 상태 없는 연산

- 연산을 처리하면서 상태값을 따로 **저장해둘 필요가 없는 연산**



## 상태 있는 연산

- 연산을 처리하면서 상태값을 따로 **저장해둬야하는 연산**
- distinct, skip, limit, sorted, reduce(최종연산)



## 쇼트서킷

- 결과를 찾는 즉시 반환한 후, 전체 스트림을 처리하지 않는다.
- and 연산 또는 or연산으로 연결된 표현식이 있다고 했을 때 하나라도 거짓인 결과가 나오면 나머지 표현식의 결과와 상관없에 전체 결과를 거짓으로 본다. 이러한 상황을 **쇼트 서킷**이라 한다.
- stream에서 대표적으로 최종연산 allMatch, noneMatch, findFirst, findAny 등이 있다. 중간연산에서는 limit도 쇼트서킷 연산으로 처리된다.



## 필터링

- **filter**를 이용한 필터링

  ![](./img/filter.PNG)

  ```java
  menu.stream()
      .filter(m -> m.getCal > 30) // 칼로리가 30이상인 메뉴만 필터링
      .collect(toList());
  ```

- **distinct**를 이용한 필터링

- **limit(n)**를 이용한 필터링(축소) - 쇼트서킷 포함



## 슬라이싱

- **skip(n)**을 이용해서 n개 요소를 제외한 스트림을 반환하는 슬라이싱기법



## 매칭

- 특정 객체에서 특정 데이터를 선택하는 기능을 제공한다.

- filter를 통해서도 특정객체의 데이터에 접근이 가능하지 않느냐고 한다면

   filter같은 경우에는 해당하는 조건의 객체 자체를 가져와서 스트림으로 반환하지만,

   map을 사용하면 해당 객체의 멤버변수를 새로운 스트림으로 변환하여 새로운 버전으로 만든다.

- **map을 이용한 매칭**

```java
menu.stream()
    .map(Dish::getName) /*Dish를 스트림으로 변환하는 것이 아닌 Dish의 name을 스트림으로 반환한다.
    					  따라서 Stream<String>가 반환된다.*/
    .collect(toList());
```



- **flatMap을 이용한 매칭**

```java
menu.stream()
    .map(w -> w.split(""))//Stream<String[]> 형식으로 반환된다.
    .flatMap(Array::stream) //Stream<String> 형식으로 변환해준다.
    .distinct() //상태있는 중간연산임. p155 5-6참고
    .collect(toList());
```



## 검색 - 쇼트서킷 평가를한다.

- 전체를 검색하는 allMatch
- 적어도 한 요소와 일치하면 anyMatch
- 일치하는 요소가 하나도 없으면 noneMatch

=> 3가지 연산 모두 boolean값을 리턴한다.

- 현재 스트림에서 해당 요소를 반환하는 findAny -> 조건에 맞는 요소가 있으면 즉시 실행을 종료한다.



## 리듀싱

- 모든 스트림 요소를 반복적으로 처리해서 값으로 도출하는 메소드다.

  ![](./img/reduce.PNG)

  -초기값을 주고 초기값과 연산함수를 통해 결과를 축적한다.



## Optional

- 값이 존재하는지 확인하고 값이 없을 때 어덯게 처리할 것인지에 대해서 강제한다



## iterate, generate

- iterate, generate를 사용하면 기존의 고정된 컬렉션이 아닌 무한한 크기의 스트림을 만들수있다.

![generate와 iterate](./img/iterate,generate.PNG)

#### #generate 사용법

- 보통 generate는 생산된 각 값을 연속적으로 계산하지 않는다. 이점이 iterate와 차이가 있다.

```java
Stream.generate(Math::random)
      .limit(5) //무한하게 늘어나기때문에 limit로 제한을 둬야한다.
      .forEach(System.out::println);
```



#### #iterate 사용법

- 일반적으로 연속된 일련의 값을 만들 때 주로 사용된다.

```java
Stream.iterate(0, n -> n+2)
      .limit(10)
      .forEach(System.out::println);
```





------



##### 참고링크

[자바성능비교 TOP 10]: https://blog.jooq.org/2015/02/05/top-10-easy-performance-optimisations-in-java/
[스트림사용의 10가지 주의사항]: https://okky.kr/article/329818



[^1]:  레이스 컨디션은 공유 자원에 대해 여러 개의 프로세스가 동시에 접근하기 위해 경쟁하는 상태를 말한다.

