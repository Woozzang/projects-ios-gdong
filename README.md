# [GDONG_log](https://github.com/gongddong/GDONG-front)

* **x86 컴퓨터로 작성한 프로젝트**

  m1 (arm-64) 으로로 돌리려고 하니 오류가 뜸

  Could not find simulator for target 'x86_64-apple-ios-simulator'; found: arm64, arm64-apple-ios

  [링크](https://stackoverflow.com/questions/56957632/could-not-find-module-for-target-x86-64-apple-ios-simulator)

  ***뭔가 진짜 알수없는 오류들이 엄청 생겼는데,***  pod 을 한번 싹 밀고 다시 `pod install`하니깐 해결되었다.



* **hasValidValue, isValid 속성들의 삭제**

  위 속성들이 필요하다고 생각한 이유: '완료' 버튼을 눌러서 글을 서버에 포스트 하는 시점에, 셀에 유효한 값들이 들어있는지 확인

  그런데 버튼을 누르는 시점에 한번만 판단하면 되므로 속성을 통해 지속적으로 관리할 필요는 없다.

  

  또한,  관리하려면 어떤 Delegate method 에 코드가 추가되어야 되는지 못찾음

  "textFieldDidendEditing" : 이 메서드는 textField가 더 이상 `firstResponder`가 아닌 시점에 호출됨

  

  이것으로 하려고 하였으나 firstResponder 를 resign 되는 시점이 정확하지 않았다.

  textField 에서 다른 textField 로 이동할 때는 `resign` 이 됨

  그런데 textField에 focus 된 상태에서 '완료' 버튼을 눌르면 firstResponder가 `resign` 되지 않음

  '더 정확히는 becomeFirstResoponder 를 호출해도 false 가 리턴됨'

  button을 firstResponder로 만드는 메서드를 호출해도 resign 되지 않음

  

  **!!!방법 찾아냄!!!**

  해당 textField 를 속성으로 가지고 있다가 resignFirstResponder() 호출해야함 (textFieldDidEndEditing 도 호출되었음)

  textField가 여러개일때는 모두 가지고 있어야 resgin 이 가능할 것임

  

  "textFieldShouldReturn" : 이것은 return을 누를 때 호출되는데, return을 누르지 않고 완료를 누를 수 있기 때문에 여기도 공백이 생김



* ~~CustomCell .xib 로 구현~~ (구현 완료)

* 테이블뷰 셀 selection 비활성화

  



## Bar Button Item

* 컨트롤 드래그를 통한 IBAction 연결 유효한지 확인해보자

​	도큐먼트 아웃라인에  `I` 라고 적혀있는 것을 컨트롤 드래그해서 연결했는데 제대로 동작한다.



## TextField

* TextField FirstResponder resign 하는 시점

  확실한 것: textfield -> textField 는 resign 됨

  textField -> button 은 resign 안됨



## NumberFormatter

* 동적으로 3자리수마다 컴마 추가하기

https://stackoverflow.com/questions/27308595/how-do-you-dynamically-format-a-number-to-have-commas-in-a-uitextfield-entry



## TODO

* # textField(_:shouldChangeCharactersIn:replacementString:) 메서드 이해하기





## 💡 알게 된 것 (feat. 사소한 것까지)

* 코드에서 해당 view 가 `onscreen` 상태인지 확인하는 법

  window 속성이 nil 이 아니면 `onscreen` 이다.

  

* tableView 에서 'No Selection' 상태를 하면 didSelectRowAt 이 호출되지 않는다.

  selection 상태의 색을 투명으로 하고 싶어서 생각한 것이었는데, 색을 바꾸는 방법이 존재했다. (아래에 적어놓음),

  

* Documentation 에서 발견하긴 하였는데, UIStoryboardSegue 를 직접 생성하지 말라고 적혀있었다.

  물론 이 타입의 인스턴스 메서드인 performSegue() 는 programmatically 하게 사용할 수 있다.



* ❗️❗️❗️ XCode의 project navigator 에서 `discard all changes` **절대절대절대절대** 하지말자

  이거 누르고 복구하느라 3시간 걸렸다.

  적어도 XCode 사용법을 완벽하게 이해하기 전까지는 하지말자.

  

## ❓ 궁금한 점과 해결

*  `.xib` 로 구현한 `커스텀 cell` 의 서브 뷰들의 delegate 지정 시점은 언제일까?

  `storyboard` 에서 프로토타입 cell을 구현했으면 `컨트롤 드래그`로 해결되는데, .xib 로 구성한 cell 은 코드로 구현해야 한다.

  나의 해결법

  1. delegate 오브젝트 ( 보통 VC ) 에서 `weak var cell: UITableViewCell?(예시. 보통 CustomType이 될것임)` 속성을 추가 해 놓는다.

  2. `cellForRowAt` 에서 cell 의 identifier를 식별해내어서 참조를 얻고, 1에서 추가해 놓은 `cell` 속성에 할당하여 사용하면 된다. 예를들어 textView 의 delegate 로 VC 를 설정하고 싶은 거였다면

     ```swift
     (cellForRowAt 메서드 내부에서)
     ...
     cell.textView.delgate = self
     ...
     return cell
     ```

​		💡**여기서 느낀점**

​			cell의 재사용을 위해서 `.xib` 로 구현해 보았는데, `storyboard` 방식보다 고려해야할게 많다.

​			



* selection 색상 변경

  `storyboard` : UITableViewCell 타입 오브젝트의 Attribute Inspector 에서 None, blue, gray, default 설정 가능

  추가적으로 None 아니면 defualt (= blue, gray) 로 나뉘고 default,blue 는 gray 효과임

  `programmatically`

  ```swift
  // cell 에서  
  override func awakeFromNib() {
    super.awakeFromNib()
    
    self.selectionStyle = .default
    //self.selectionStyle = .none 이면 셀렉션 색상이 투명이다
  
    self.selectedBackgroundView = {
      let view = UIView()
      view.backgroundColor = .red
      return view
    }()
  }
  ```

  

* XCode 자체가 사용법이 매우 어렵다

  디렉터리가 실제 디렉터리랑 다르게 보이는 이유는 무엇일까?

  예를 들어 실제 디렉터리에는 9개의 파일이 있는데, XCode 에서는 3개만 보인다.



##  프로젝트 회의

* 데이터 모델 완성하기
* messageKit 학습하여 사용법 알아오기

