# 미션 - 숫자 야구 (프리코스 1주차)
## 기능 요구사항

- **(1) 게임 시작:** 컴퓨터는 무작위 3자리 숫자를 생성한다.
  - 각 숫자는 1~9의 수 중 하나이다.
  - 각 자리의 숫자는 서로 달라야 한다.
- **(2) 사용자 입력:** 사용자는 콘솔에 3자리 숫자를 입력한다.
  - 사용자가 잘못된 값을 입력하는 경우, `IllegalArgumentException`을 발생시킨 후 애플리케이션을 종료한다.
- **(3) 결과 출력:** 컴퓨터는 입력한 숫자에 대한 결과를 출력한다.
  - 이때, 정답을 맞힌 경우와 맞히지 못한 경우로 나누어 처리한다.
  - **(3-1) 정답을 맞힌 경우**
    - 게임 종료 문구를 출력하고, 사용자의 입력을 기다린다.
    - 사용자가 '1'을 입력하는 경우, (1) 게임 시작 과정으로 돌아간다.
    - 사용자가 '2'를 입력하는 경우, (4) 게임 종료 과정으로 이동한다.
    - 사용자가 잘못된 값을 입력하는 경우, `IllegalArgumentException`을 발생시킨 후 애플리케이션을 종료한다. 
  - **(3-2) 정답을 맞히지 못한 경우**
- **(4) 게임 종료:** 애플리케이션을 완전히 종료한다.
  - 사용한 리소스는 모두 해제한다.

## 예외 상황
"사용자가 잘못된 값을 입력하는 경우"에 해당하는 예외 상황을 정리하였습니다.
  - 입력이 null인 경우
  - 숫자가 아닌 값을 포함하는 경우
  - 같은 수를 여러개 포함하는 경우
  - 숫자 중 0을 포함하는 경우
  - 이 경우 `IllegalArgumentException`을 발생시킨 후 애플리케이션을 종료한다.

## 미션 수행 중 생각 정리
미션을 진행하며 생각한 내용을 정리합니다. 개발을 진행하며 지속적으로 업데이트 하였습니다.
- 실제 구현에 들어가기 앞서, 요구사항을 정확하게 파악하려 노력하였습니다. 먼저 기능 요구사항과 각각의 기능 구현 중 발생할 수 있는 예외 케이스를 `docs/README.md` 파일에 정리하였습니다.
- 미션에서 제공하는 `camp.nextstep.edu.missionutil` 패키지에 포함된 메서드들의 동작을 파악하기 위해 힘썼습니다.
  - 각 메서드의 내부 구현 코드를 살펴보았고, 값을 넣어보며 예상대로 동작하는지 검증 해보았습니다.
- magic number의 처리
  - 숫자를 하드코딩 하지 않고, 어떻게 의미있는 이름을 가진 상수로 처리할지 고민하였습니다.
  - 각 클래스마다 상수를 선언하기에는 코드의 가독성이 떨어지게 된다고 생각하였고, 따라서 상수를 constants 패키지로 따로 분리하고 import하여 사용하는 방식으로 변경
- 컴퓨터가 정답을 생성할 때 중복을 어떻게 처리할지 고민하였습니다.
  - 처음에는 아래와 같이 중첩 반복문을 돌 때마다 지금까지 생성한 숫자들과 비교하는 방식으로 코드를 작성하였습니다.
  - 그러나 입력이 잦은 상황에서 매번 이와 같은 로직을 수행하는 것은 비효율적이라고 생각했고, 수정의 필요성을 느꼈습니다.
  - 그때 순서를 유지하며 중복을 허용하지 않는 자료구조인 `LinkedHashSet`에 대해 알게되었고, 이를 코드에 적용하여 중복을 제거하는데에 사용하였습니다.
    - **처음 작성한 코드**
    ```
        public void generateAnswers() {
          for (int i = 0; i < 3; i++) {
              int candidateNum;
              do {
                  candidateNum = Randoms.pickNumberInRange(1, 9);
              } while (checkDuplicateNum(candidateNum));
              generatedAnswers.add(candidateNum);
          }
      }

      // 무작위로 생성한 숫자가 정답 숫자 리스트에 포함되어 있는지 검사
      private boolean checkDuplicateNum(int candidateNum) {
          if (generatedAnswers.contains(candidateNum)) {
              return true;
          }
          return false;
      }
    ```
    - **수정한 코드**
    ```
    public static List<Integer> generateAnswers() {
          // LinkedHashSet을 활용하여 중복 제거
          Set<Integer> generatedAnswers = new LinkedHashSet<>();

          while (generatedAnswers.size() < Constants.CORRECT_ANSWER_LENGTH) {
              int candidateNum = Randoms.pickNumberInRange(Constants.MIN_GUESS_NUMBER, Constants.MAX_GUESS_NUMBER);
              generatedAnswers.add(candidateNum);
          }
          return new ArrayList<>(generatedAnswers);
      }
    ```

- 역할 분리
  - 처음에는 `BaseballGame` 클래스에 거의 모든 코드가 들어가 있었습니다. 그러나 코드가 길어짐에 따라 기능 수정 시 하나의 파일을 뒤적이며 해당 코드를 찾아내는데 오랜 시간이 걸렸고, 이는 유지보수에 어려움이 되었습니다.
  - 또, 역할을 분리하지 않았기 때문에 복잡한 코드들이 서로 의존하고 있었습니다. 이는 테스트코드를 작성함에 있어 매우 큰 장애물이었습니다.
  - 따라서 다음과 같이 별도의 클래스를 생성하여 `BaseballGame` 클래스가 담당하던 역할을 분리하였습니다.
    - `AnswerGenerator` : 컴퓨터의 정답을 생성
    - `Convertor` : 출력을 위해 list를 String으로 변환
    - `Judge` : 각 입력에 대한 Score 및 정답 여부를 판단
    - `UserInputReceiver` : 사용자의 입력을 처리
    - `Validator` : 사용자의 입력이 유효한지 검증
- 클래스와 메서드, 변수의 이름을 어떻게 작성할 것인지 고민하였습니다.
  - 호출한 메서드 이름만 보고도 어떤 동작을 수행하는지 고민해보았습니다.
  - 영단어 사전과 번역기를 사용하여 상황에 적절한 단어를 선택하기 위해 노력했습니다.
  - 비슷한 동작을 하는 메서드들의 이름은 어떻게 해야 통일성을 지킬 수 있을지 고민하였습니다.
    - 예) `Validator` 클래스의 `check...()` 메서드들
- 어떻게 해야 가독성이 좋고, 효율적으로 동작하는 코드를 작성할 수 있을지 고민하였습니다.
  - 기존에 작성한 코드들을 stream을 사용하여 가독성을 높이고, 깔끔한 코드를 유지하기 위해 

### 개선할 점
- 역할을 분리하려고 노력하였으나, 제대로 분리하지 못하여 각 클래스들이 서로 의존하는 문제가 있었습니다.
  - 강한 의존성으로 인해 동작 흐름이 각 클래스를 복잡하게 넘나드는 스파게티 코드가 되어버렸고, 유지보수에 어려움을 겪었습니다.
  - 메서드들이 서로 얽혀있어 동일한 매개변수를 여러번 반복해서 전달하는 문제도 발생했습니다.
    - 메서드A로 전달한 매개변수들을 메서드A 내부에서 호출한 메서드B로 전달하고, 다시 메서드B에서 호출한 메서드C로 전달하는 다음과 같은 모양이 되어버렸습니다.
      ```
      methodA(param1, param2) {
          methodB(param1, parma2) {
              methodC(param1, param2) {
                  ...
              }
          }
      }
      ```
    - 이와 같은 이유로, 파라미터를 수정할 일이 생기면 모든 메서드의 파라미터를 수정해야 하는 현상까지 발생하였습니다.
  - 출력문이 여기저기 복잡하게 분포되어있어 로직의 순서를 변경하면 출력문의 순서 또한 변경되어버리는 문제 또한 겪었습니다.

### 느낀점
처음에는 간단하게 보이던 미션이었습니다. 그러나 다양한 요구사항과 테스트코드 작성 및 설계를 신경쓰면서 진행해보니, 상당히 복잡해지고 고민해야할 부분도 많다는 것을 느꼈습니다.

과제를 진행하며 부족한 점이 참 많다는 것을 느꼈습니다. 더 나은 설계를 위해 고민도 많이 하고, 더 효율적인 코드 작성을 위해 노력했지만 모자란 점 투성이었습니다. 기능을 추가할 때마다 복잡해지는 코드를 보며, 역할의 분리와 의존성 제거를 위한 객체지향적 설계의 필요성을 절실히 느끼게되었습니다.

그러나 능동적으로 문제를 해결해보고, 고민하며 지속적으로 리팩토링하는 일련의 과정은 힘들기보다는 즐겁고 뿌듯했습니다. 
이것이 우아한테크코스만의 매력이 아닐까 생각했습니다. 개발자가 교육과정에 능동적으로 참여하여 스스로의 힘으로 성장할 수 있도록 발판을 마련해주고, 진심으로 즐길 수 있도록 해주는 것 같습니다.

제가 느꼈던 부족함이 오히려 더 나은 개발자가 되기위한 동기부여가 된 것 같습니다. 
저는 앞으로 남은 프리코스를 진행하며, 점점 성장하는 모습을 보여드리고 싶습니다. 프리코스가 마무리될 즈음에는 제가 걸어온 길을 뒤돌아봤을 때 후회없이 노력했음을 스스로에게 증명하고 싶습니다.

감사합니다.