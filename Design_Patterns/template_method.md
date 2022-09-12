# Template method pattern

## 정의 

템플릿 메서드 패턴이란, 전체적인 구조는 동일하게 잡고, 특정 작업을 처리하는 일부분을 서브 클래스에서 구현하도록 한다. 확장해서 사용할 추상 클래스를 만들어서 공통으로 사용할 기능들은 구현해두고, 각 구현체(서브 클래스)마다 다르게 적용할 기능은 추상 메소드 또는 추상 클래스로 만든다. 따라서, 서브 클래스에서는 특정 기능을 재정의할 수 있다.

<hr>

## 구현 예시

```java
public abstract class Game {
   abstract void initialize();
   abstract void startPlay();
   abstract void endPlay();

   //template method
   public final void play(){

      //initialize the game
      initialize();

      //start game
      startPlay();

      //end game
      endPlay();
   }
}

public class Cricket extends Game {

   @Override
   void endPlay() {
      System.out.println("Cricket Game Finished!");
   }

   @Override
   void initialize() {
      System.out.println("Cricket Game Initialized! Start playing.");
   }

   @Override
   void startPlay() {
      System.out.println("Cricket Game Started. Enjoy the game!");
   }
}

public class Football extends Game {

   @Override
   void endPlay() {
      System.out.println("Football Game Finished!");
   }

   @Override
   void initialize() {
      System.out.println("Football Game Initialized! Start playing.");
   }

   @Override
   void startPlay() {
      System.out.println("Football Game Started. Enjoy the game!");
   }
}

public class TemplatePatternDemo {
   public static void main(String[] args) {

      Game game = new Cricket();
      game.play();
      System.out.println("________________");
      game = new Football();
      game.play();
      System.out.println("________________");	
   }
}
/*
Cricket Game Initialized! Start playing.
Cricket Game Started. Enjoy the game!
Cricket Game Finished!
________________
Football Game Initialized! Start playing.
Football Game Started. Enjoy the game!
Football Game Finished!
________________
*/
```
<hr>

## 장단점
- 장점
  - 공통으로 사용하는 코드는 추상 클래스에 구현되어있으므로 코드 중복을 줄일 수 있다. 
  - 서브 클래스에서 특정 기능을 재정의할 수 있으므로 유연성을 갖는다. 


- 단점
  - 템플릿 메서드 패턴의 흐름을 이해하거나 디버깅하는 것이 어려울 수 있다. 때로는 구현하지 말아야 할 것을 구현하거나 구현해야 할 것을 구현하지 않는 문제가 발생할 수 있다. 따라서 문서화와 엄격한 에러 처리가 이루어져야 한다. 
<hr>
