### **🛠️ 준비 사항 (Preliminaries)**

  

Scala에서 라이브러리나 프로그램을 컴파일할 때는 scalac이라는 Scala 컴파일러를 사용합니다. 이는 **Scala 3 Book**에 문서화되어 있다
```
@main def hello() = println("Hello, World!")
```

```
$ scalac hello.scala
$ scala hello
Hello, World!
```
하지만 이렇게 scalac을 직접 호출하는 방식은 소스 파일을 하나하나 넘겨줘야 해서 굉장히 번거롭고 시간이 많이 걸린다.

  

게다가 대부분의 실제 프로그램은 외부 라이브러리에 의존하고, 그 라이브러리들도 또 다른 라이브러리에 의존하는 **전이적 의존성**이 존재한다.

Scala 생태계에서는 상황이 더 복잡하다.

Scala 2.12, 2.13, 3.x 같은 여러 버전이 존재하고, JVM뿐 아니라 JavaScript, Native 플랫폼도 함께 다루기 때문이다.

  

그래서 단순히 JAR 파일과 scalac만으로 빌드를 구성하려면 반복적인 수작업이 많이 필요하다.

이러한 비효율을 줄이기 위해, 우리는 **서브 프로젝트라는 추상화**와 함께, **빌드 도구**를 도입하는 게 좋다.

### **🧰 sbt**

  

**sbt**는 Scala와 Java를 위한 **간단한 빌드 도구(Simple Build Tool)**다.

서브 프로젝트를 정의하고, 다양한 의존성과 사용자 정의 태스크(task)를 설정할 수 있다.

이를 통해 빠르고 일관성 있는 빌드를 보장할 수 있다.

  

sbt는 이런 목표를 위해 다음과 같은 기능들을 제공한다:

1. project/build.properties 파일에서 **sbt 자체의 버전**을 명시한다.
    
2. build.sbt 파일에 Scala 버전과 프로젝트 설정을 선언할 수 있는 **전용 DSL(Domain Specific Language)**을 제공한다.
    
3. 의존성 관리를 위해 **Coursier**를 사용해 필요한 라이브러리와 그 전이 의존성까지 가져온다.
    
4. **Zinc**를 통해 Scala와 Java 코드를 **증분 컴파일**한다.
    
5. 가능하면 **작업들을 병렬로 실행**해 빌드 속도를 높인다.
    
6. Maven 저장소와 호환되도록, 패키지 배포 방식에 대한 **표준 규칙**을 정의한다.
    

  

이러한 구조 덕분에 sbt는 프로그램이나 라이브러리를 빌드할 때 필요한 명령어를 **표준화**하고,

JVM 생태계와의 연동도 자연스럽게 처리할 수 있도록 도와준다.

### **💡 왜 build.sbt DSL을 사용하는가?**

  

sbt를 특별하게 만드는 이유 중 하나는 바로 build.sbt DSL이다.

다른 빌드 도구들이 YAML, TOML, XML 같은 설정 파일 형식을 사용하는 반면,

sbt는 자체적으로 정의한 **도메인 특화 언어(DSL)**를 사용한다.

  

이 DSL은 2010년부터 2013년 사이에 개발되었고, 처음에는 단순하게 scalaVersion과 libraryDependencies만 선언하는 구조로 시작할 수 있어서

겉보기에는 YAML이나 JSON처럼 보이기도 한다.

하지만 프로젝트 규모가 커질수록, 이 DSL은 더 강력한 기능을 통해 **구성 정리와 확장성**을 확보할 수 있게 해준다.

  

#### **🔧 DSL이 제공하는 주요 기능은 다음과 같다:**

- **중복 제거**: 동일한 라이브러리 버전 같은 정보를 반복해서 쓰지 않기 위해, val 키워드로 변수를 선언할 수 있다.
    
- **조건부 설정**: 필요할 경우 if 같은 Scala 문법을 활용해 설정이나 태스크를 조건부로 정의할 수 있다.
    
- **정적 타입 지원**: 설정 값이나 태스크는 정적 타입을 가지고 있어서, 오타나 타입 오류를 빌드 시작 전에 잡아낼 수 있다.
    
    또한 타입 정보는 태스크 간에 데이터를 전달할 때도 유용하게 작용한다.
    
- **직관적인 태스크 그래프 구성**: Initialized[Task[A]]를 통해 구조적인 동시성(concurrency)을 제공한다.
    
    DSL에서는 .value 문법을 사용해 간결하고 명시적으로 태스크 간 의존 관계를 정의할 수 있다.
    
- **플러그인 확장성**: 커뮤니티는 sbt 플러그인을 통해 새로운 태스크나 언어 확장을 추가할 수 있다.
    
    예를 들어 Scala.js, sbt-native-packager 같은 플러그인들이 존재한다.

### **🧩 sbt 구성 요소 (sbt components)**

  

#### **🔸 sbt 러너 (sbt runner)**

  

sbt 빌드는 **sbt runner**를 통해 실행된다.

이 sbt runner는 다른 구성 요소들과 구분하기 위해 “sbt-the-shell-script”라고도 불린다.

여기서 중요한 점은 **sbt runner는 어떤 버전의 sbt도 실행할 수 있도록 설계되었다**는 점이다.

  

#### **🔸**  **project/build.properties** **로 sbt 버전 지정하기**

  

sbt runner는 내부적으로 **sbt launcher**라는 하위 구성 요소를 실행한다.

이 launcher는 project/build.properties 파일을 읽어서, 어떤 sbt 버전을 사용할지 결정한다.

그리고 해당 버전이 로컬에 캐시되어 있지 않으면, 관련 아티팩트를 자동으로 다운로드한다.

```
sbt.version=2.0.0-M3
```
이런 구조 덕분에 다음과 같은 장점이 있다:

1. 누군가가 내 프로젝트를 클론(clone)해서 빌드를 실행하더라도,
    
    그 사람이 어떤 sbt runner를 설치했는지와 상관없이 **정해진 sbt 버전이 사용된다**.
    
2. sbt 버전의 변경 내역은 Git 같은 **버전 관리 시스템에 기록될 수 있다.**
    

---

이 구조는 팀 개발이나 오픈소스 프로젝트에서 **환경 차이를 최소화하고, 재현 가능한(reproducible) 빌드 환경을 만드는 데 매우 유용**하다.

### **⚙️ sbtn (sbt --client)**

  

sbtn은 sbt runner의 하위 구성 요소이며, --client 플래그를 sbt 실행 시 넘기면 호출되는 **네이티브 얇은 클라이언트(native thin client)**다.

이 클라이언트는 **sbt 서버에 명령을 보내는 역할**을 한다.

  

sbtn이라는 이름은 **GraalVM의 native-image** 기능을 사용해 컴파일된 **네이티브 실행 파일**이라는 점에서 유래되었다.

  

sbtn과 sbt server 사이의 통신 프로토콜은 상당히 안정적이라, **대부분의 최신 sbt 버전 간에도 호환성 문제 없이 작동한다.**

---

### **sbt server ( sbt --server )**

  

sbt server는 실제 빌드 작업을 수행하는 핵심 도구다.

이 서버의 버전은 project/build.properties에 명시된 내용을 기반으로 결정된다.

  

sbt server는 **캐셔(cashier)**처럼 동작하면서,

sbtn이나 코드 에디터(예: IntelliJ, VS Code 등)로부터 들어오는 명령을 받아 처리한다.

---

### **📦 Coursier**

  

sbt server는 내부적으로 **Coursier**라는 구성 요소를 실행해서 다음과 같은 의존성을 해결한다:

- Scala 표준 라이브러리
    
- Scala 컴파일러
    
- 사용자가 설정한 외부 라이브러리들
    

  

Coursier는 빠르고 신뢰성 높은 의존성 관리 도구이며,

sbt가 Scala 생태계에서 강력한 의존성 해결 능력을 갖추도록 해준다.

## build.sbt 작성법

### **✅ 1.** **Key-Value 설정 (Setting Expressions)**

```
scalaVersion := "3.3.5"
name := "my-app"
```
- := 연산자는 **설정 값을 지정**할 때 사용한다.
    
- 좌측의 scalaVersion, name 같은 키는 SettingKey 또는 TaskKey 타입이다.

### **✅ 2.**  **의존성 선언 (libraryDependencies)**

```
libraryDependencies += "org.typelevel" %% "cats-effect" % "3.5.4"
```
- += : 단일 항목 추가
    
- ++= : 여러 항목 추가 (Seq(...) 형태로)

```
libraryDependencies ++= Seq(
  "org.typelevel" %% "cats-effect" % "3.5.4",
  "org.scalatest" %% "scalatest" % "3.2.17" % Test
)
```
- % 하나: 직접 아티팩트 이름 지정 (cats-effect_3)
- % 두 개: Scala 버전에 맞는 artifact 자동 매칭 (cats-effect_2.13 등)

### **✅ 3.**  **val, lazy val 선언**

```
val commonVersion = "0.2.0"
lazy val root = (project in file(".")).settings(
  name := "my-app",
  version := commonVersion
)
```

- val: 일반 상수 (즉시 평가)
    
- lazy val: **지연 평가**. sbt에서는 순서 문제나 캐싱에 유리하다.

### **✅ 4.** **.settings(...) 블록**

```
lazy val core = (project in file("core"))
  .settings(
    name := "core",
    scalaVersion := "3.3.5"
  )
```

- .settings(...) 안에는 설정 키와 값들을 나열할 수 있다.
    
- 보통 lazy val로 서브 프로젝트를 정의할 때 함께 사용한다.

### **✅ 5.**  **플러그인 설정 (plugins.sbt)**

```
// project/plugins.sbt
addSbtPlugin("org.scalameta" % "sbt-scalafmt" % "2.5.0")
```

- project/ 디렉토리 안의 plugins.sbt에서 플러그인을 추가할 수 있다.

### **✅ 6.**  **작업(Task) 정의와 실행**

```
val hello = taskKey[Unit]("Prints hello")
hello := { println("Hello!") }
```

- taskKey[T]("설명")  으로 사용자 정의 태스크 생성 가능
    
- := 오른쪽에는 코드 블록을 사용한다
    
- sbt shell에서 hello 명령어 입력으로 실행할 수 있다


### **✅ 7.**  **조건부 설정 (Scala 문법 활용)**

```
scalacOptions ++= {
  if (scalaVersion.value.startsWith("3"))
    Seq("-Xfatal-warnings")
  else
    Seq("-deprecation", "-unchecked")
}
```

- build.sbt는 Scala 기반이라 if, val, match 같은 일반적인 문법도 사용 가능하다.

### **✅ 8.**  **import 사용하기**

```
import Dependencies._
libraryDependencies += toolkit
```

- 외부 정의(project/Dependencies.scala)에 있는 설정을 사용할 때 import를 활용한다.
    
- 일반 Scala처럼 import가 가능하다.
### **빌드 정의란 ?**


**빌드 정의(build definition)는 build.sbt 파일 안에서 정의되며,

하나 이상의 **프로젝트(Project 타입의 인스턴스)**로 구성된다.

하지만 “프로젝트(project)”라는 용어는 혼동을 줄 수 있기 때문에, 이 문서에서는 보통 **서브프로젝트(subproject)**라는 표현을 사용한다.

  

예를 들어, 현재 디렉토리에 있는 서브프로젝트를 build.sbt에서 다음과 같이 정의할 수 있다:
```
scalaVersion := "3.3.3"
name := "Hello"
```
혹은 좀 더 명시적으로 다음과 같이 작성이 가능하다.
```
lazy val root = (project in file("."))
  .settings(
    scalaVersion := "3.3.3",
    name := "Hello",
  )
```

각 서브프로젝트는 **key-value 쌍**으로 설정된다.

lazy val root : root라는 느리게 평가되는 값을 선언하며 이 값이 서브프로젝트를 나타낸다.

(project in file(".")) : 현재 디렉토리에 위치한 서브프로젝트를 초기화한다.

예를 들어 name이라는 키는 문자열 값을 가지며, 이 값이 서브프로젝트의 이름이 된다.


이러한 key-value 쌍은 .settings(...) 메서드 안에 나열된다.

이 방식은 빌드 정의를 **유연하고 명확하게 구성**할 수 있게 해준다.

### **Typed Setting Expression**

sbt의 build.sbt DSL을 좀 더 자세히 살펴보면 다음과 같은 구조로 되어 있다:
```
organization  :=         "com.example"
^^^^^^^^^^^^  ^^^^^^^^   ^^^^^^^^^^^^^
   키          연산자         값(body)
```
좌변에는 name, version, scalaVersion 같은 key가 있다.

키는 SettingKey[T], TaskKey[T] 또는 InputKey[T] 의 인스턴스이며 T는 기대되는 값의 유형이다.

ex) name키는 SettingKey[String] 으로 타입이 지정되어 있기 떄문에 name에 대한 := 연산자는 String으로 구체화된다.

build.sbt 파일은 val, lazy val, def 등이 쓰일 수 있지만 상위 수준의 객체 및 클래스는 허용되지 않으며 이런 내용들은 project/ 디렉토리에 있는 Scala 소스 파일로 이동해야한다.

+= 연산자를 통해서 value를 추가할 수도 있다.


### **val과 lazy val의 활용**
라이브러리 버전 번호 같은 정보를 반복해서 쓰지 않기 위해, build.sbt 안에는 val, lazy val, def 등을 함께 사용할 수 있다.

```
val toolkitV     = "0.2.0"
val toolkit      = "org.scala-lang" %% "toolkit" % toolkitV
val toolkitTest  = "org.scala-lang" %% "toolkit-test" % toolkitV

scalaVersion := "3.6.4"
libraryDependencies += toolkit
libraryDependencies += (toolkitTest % Test)
```

## Keys
키에는 3가지 유형이 있다.
- SettingKey[T] : 값이 한 번만 평가되는 키이다. 즉, 서브프로젝트를 로드할 때 값이 계산되고, 이후에 유지된다.
- TaskKey[T] : 값에 대한 키로, 이 키는 호출될 때마다 (Scala 함수처럼) 평가된다. 이 과정에서 부작용이 발생할 수 있다.
- InputKey[T] : 명령줄 인수를 입력으로 받는 작업을 위한 키이다. 이에 대한 상세 내용은 Input Tasks를 참조하면 된다.

### 내장 키 
내장 키는 Keys 라는 객체의 필드로 정의된다. build.sbt 파일은 암묵적으로 import sbt.Keys._ 를 포함하고 있으므로 sbt.Keys.name을 name으로 간단히 참조할 수 있다.

### Custom Keys
커스텀 키는 각각의 생성 메서드인 settingKey, taskKey, inputKey를 사용하여 정의할 수 있다.

```
lazy val hello = taskKey[Unit]("An example task")
```

## Bare .sbt build definition
설정은 .settings(...) 호출 대신 build.sbt 파일에 직접 작성할 수 있다. 
```
ThisBuild / version := "1.0
ThisBuild / scalaVersion := "2.12.18"
```

### **libraryDependencies**  Key

외부 라이브러리를 추가하려면, 다음과 같이 libraryDependencies 키를 사용해서 선언한다.

여기서 groupId, artifactId, revision은 모두 문자열이다:

```
libraryDependencies += groupID % artifactID % revision
```

## Multiple subprojects
subproject들이 서로 의존하고 함께 수정되는 경향이 있을 때 이것들을 하나의 build에 유지하는 것이 유용하다.

각 subproject들은 자체 소스 디렉토리를 갖고 있으며 package 명령을 실행할 때 자체 JAR 파일을 생성한다.

```
lazy val util = (project in file("util"))

lazy val core = (project in file("core"))
```

### 빌드 전역 설정
여러 subproject에 걸쳐 공통 설정을 추출하기 위해 ThisBuild에 스코프를 설정하여 정의할 수 있다.

ThisBuild는 빌드의 기본값을 정의하는 데 사용할 수 있는 특별한 subproject 이름이다.

하나 이상의 subproject를 정의하고 scalaVersion 키를 정의하지 않으면 sbt는 ThisBuild / scalaVersion을 찾게된다.

```
ThisBuild / organization := "com.example"
ThisBuild / version      := "0.1.0-SNAPSHOT"
ThisBuild / scalaVersion := "2.12.18"

lazy val core = (project in file("core"))
  .settings(
    // 기타 설정
  )

lazy val util = (project in file("util"))
  .settings(
    // 기타 설정
  )
```
이런식으로 설정하게 되면 버전을 한 곳에서 변경할 수 이쏙 빌드를 다시 로드할 때 subproject 전반에 걸쳐 반영된다.

### 공통 설정
commonSettings seq를 통해 프로젝트에서 공통으로 사용하는 설정을 추출할 수 있다.
```
lazy val commonSettings = Seq(
  target := { baseDirectory.value / "target2" }
)

lazy val core = (project in file("core"))
  .settings(
    commonSettings,
    // 기타 설정
  )

lazy val util = (project in file("util"))
  .settings(
    commonSettings,
    // 기타 설정
  )
```
위 코드에서 commonSettings는 core와 util 모두에 적용된다. 이를 통해 중복 코드를 줄이고 일관된 설정을 유지할 수 있다.

## Scope
특정 설정이나 작업이 적용되는 범위를 의미한다. 예를 들어 compile이나 test 같은 키가 있다.
- key : 설정이나 작업을 정의할 때 사용되는 식별자
- build scope : sbt는 다음과 같은 4가지 주요 scope를 지원한다.
  - Global Scope : 모든 프로젝트와 설정에 적용된다. 
  - ThisBuild Scope : 현재 빌드 파일에만 적용된다. 일반적으로 멀트프로젝트 설정에서 공통적으로 사용된다.
  - Project Scope : 특정 프로젝트에만 적용된다. 여러 프로젝트가 있을 때 개별 프로젝트의 설정을 다를 수 있게 해준다.
  - Configuration Scope : 특정 구성에 해당하는 설정이다. 예를 들어 compile, test, runtime 등 각각의 환경을 위한 설정을 정의할 수 있다.
- 식별 표현 : sbt 에서는 / 연산자를 사용하여 scope를 식별한다.
```
lazy val myProject = (project in file(".")).
  settings(
    name := "My Project",
    Compile / scalaVersion := "2.13.6",   // Compile 스코프
    Test / scalaVersion := "2.13.5"        // Test 스코프
  )
```
## Task
sbt에서 Task는 특정 작업을 수행하기 위해 정의된 실행 가능한 단위이며 작업의 종류나 실행 결과에 따라 다양한 기능을 수행한다.

기본적으로 Task는 비동기적으로 실행되며 필요할 때마다 호출되며 재계산된다.

### 주요 개념
- return 값 : Task는 실행 후 결과를 반환한다. 반환타입은 정의 때 명시하며 다양한 데이터 타입이 될 수 있다.
- Lazy Evaluation : sbt의 task는 호출될 때까지 실행되지 않으며, 필요한 경우에 재계산된다. 이 특성 덕분에 불필요한 작업을 피할 수 있다.
- TaskKey : Task를 정의할 때 사용되는 객체로, Type과 Description을 설정하여 Task의 이름과 반환 타입을 지정한다.
  ```
  lazy val myTask = taskKey[Unit]("내 사용자 정의 작업")
  ```
- 의존성 : Task는 다른 Task 및 설정에 의존할 수 있다.
```
lazy val printHello = taskKey[Unit]("Hello를 출력하는 작업")

printHello := {
  val log = streams.value.log
  log.info("Hello, SBT!")
}
```

## Dependency
빌드 내의 프로젝트들은 완전히 독립적일 수 있지만 일반적으로는 dependency를 통해 서로 연결된다. dependency는 크게 2 종류로 나뉜다.

### Aggregation
```
lazy val root = (project in file("."))
  .aggregate(util, core)

lazy val util = (project in file("util"))

lazy val core = (project in file("core"))
```
.aggregate를 선언하면 root 프로젝트에 어떤 작업을 실행하면 util,core 모두에게 동일한 작업이 수행된다.

특정한 작업을 피하고 싶다면 다음과 같이 작성할 수 있다.
```
lazy val root = (project in file("."))
  .aggregate(util, core)
  .settings(
    update / aggregate := false
  )
```
### ClassPath
한 프로젝트가 다른 프로젝트의 코드에 의존하면 dependsOn 메서드 호출을 통해 구현이 가능하다.
```
lazy val core = project.dependsOn(util)
```

### Per-Configuration classpath dependencies
core dependsOn(util) 은 core의 컴파일 구성이 util 의 컴파일 구성에 의존함을 의미한다.

좀 더 명시적으로 쓰면 dependsOn(util % "compile->compile") 로 쓸 수 있다.

여기서 -> 는 의존한다는 의미이다.  

->config 를 생략할 경우 기본적으로 ->compile로 간주된다.

유용한 선언중 하나로 "test->test" 가 있는데 이는 테스트가 테스트를 의존함을 의미한다.

이 설정을 사용하면 util/src/test/scala에 테스트용 코드를 배치하고 이를 core/src/test/scala에서 사용할 수 있다.

여러 개의 구성에 대해 의존성을 정의하고 싶다면 세미클론으로 구분하여 선언할 수 있다.

```
dependsOn(util % "test->test;compile->compile")
```

## **Dependency를 한 곳에서 관리하기**
.scala 파일은 project/ 디렉토리 아래에 위치하면 **빌드 정의의 일부**가 된다.

이 점을 활용해서, dependency들을 한 곳에 모아 관리할 수 있다.

  

예를 들어 project/Dependencies.scala라는 파일을 만들고 다음처럼 작성할 수 있다:

```
// project/Dependencies.scala

import sbt.*

object Dependencies:
  // 버전 변수
  lazy val toolkitV = "0.2.0"

  // 의존성 정의
  val toolkit      = "org.scala-lang" %% "toolkit" % toolkitV
  val toolkitTest  = "org.scala-lang" %% "toolkit-test" % toolkitV
end Dependencies
```

이렇게 정의하면 build.sbt에서 이 Dependencies 객체를 사용할 수 있다.

편리하게 사용하기 위해 import Dependencies.*를 추가해주면 된다:

```
import Dependencies.*

scalaVersion := "3.6.4"
name := "something"

libraryDependencies += toolkit
libraryDependencies += toolkitTest % Test
```
이 방식은 dependency 관리의 **중복을 줄이고**,

**프로젝트 구조를 모듈화하고 유지보수하기 쉽게 만들어준다.**

### **Dependency 트리 확인하기**

```
> Compile/dependencyTree
```
이 명령어는 직접추가한 dependency 뿐 아니라 전이적으로 추가된 의존성까지 포함하여 전체 트리를 출력해준다.
```
sbt:bar> Compile/dependencyTree
[info] default:bar_3:0.1.0-SNAPSHOT
[info]   +-org.scala-lang:scala3-library_3:3.3.1 [S]
[info]   +-org.scala-lang:toolkit_3:0.2.0
[info]     +-com.lihaoyi:os-lib_3:0.9.1
[info]     | +-com.lihaoyi:geny_3:1.0.0
[info]     | | +-org.scala-lang:scala3-library_3:3.1.3 (evicted by: 3.3.1)
[info]     | | +-org.scala-lang:scala3-library_3:3.3.1 [S]
...
```

## **sbt의 Build Layout (디렉토리 구조)**

```
.
├── build.sbt
├── project/
│   ├── build.properties
│   ├── Dependencies.scala
│   └── plugins.sbt
├── src/
│   ├── main/
│   │   ├── java/
│   │   ├── resources/
│   │   ├── scala/
│   │   └── scala-2.13/
│   └── test/
│       ├── java/
│       ├── resources/
│       ├── scala/
│       └── scala-2.13/
├── subproject-core/
│   └── src/
│       ├── main/
│       └── test/
├── subproject-util/
│   └── src/
│       ├── main/
│       └── test/
└── target/
```
- **. (루트 디렉토리)**
    
    sbt에서 빌드를 시작하는 기준점이 되는 디렉토리다.
    
    일반적으로 이 디렉토리에 build.sbt가 위치한다.
    
- **build.sbt**
    
    빌드 정의를 작성하는 메인 설정 파일이다.
    
    실제로는 *.sbt 확장자를 가진 모든 파일이 적용된다.
    
- **project/**
    
    sbt 자체의 빌드 설정을 담는 디렉토리다.
    
    여기에 있는 파일들은 **sbt 실행 시 먼저 해석**된다.
    
    - build.properties: 사용하려는 sbt 버전을 지정한다.
        
    - plugins.sbt: 사용할 sbt 플러그인을 선언한다.
        
    - Dependencies.scala: 라이브러리 의존성 정보를 정리해 두는 곳이다.
        
    - .scala 파일: 헬퍼 객체나 간단한 플러그인을 정의할 수 있다.
        
    
- **src/main/**
    
    실제 애플리케이션 코드를 넣는 디렉토리다.
    
    이 안에 Java, Scala, 자원 파일 등을 각각 나눠서 관리한다.
    
- **src/test/**
    
    테스트용 코드와 자원을 넣는 디렉토리다.
    
    구조는 main과 동일하게 구성된다.
    
- **target/**
    
    sbt가 생성하는 결과물들이 저장되는 디렉토리다.
    
    컴파일된 클래스, JAR 파일, 캐시 파일, 문서 등이 여기에 저장된다.

---

### 📚 더 알아두면 좋은 심화 기능 정리

#### 1. 프로젝트 간 코드 공유
여러 서브 프로젝트에서 공통 유틸리티를 공유하려면 `dependsOn` 또는 별도의 공통 모듈로 나누는 방식이 좋습니다.

```scala
lazy val common = (project in file("common"))

lazy val serviceA = (project in file("serviceA"))
  .dependsOn(common)

lazy val serviceB = (project in file("serviceB"))
  .dependsOn(common)
```

#### 2. 테스트 프레임워크 설정
sbt는 여러 테스트 프레임워크를 지원하며, 명시적으로 지정하거나 병렬 실행 옵션 등을 조정할 수 있습니다.

```scala
testFrameworks += new TestFramework("munit.Framework")

Test / parallelExecution := false
```

#### 3. 빌드 캐시 최적화
`sbt`는 `.ivy2`, `.coursier`, `target` 등에 캐시를 남깁니다. Docker나 CI 환경에서는 다음과 같은 디렉토리를 캐시하면 유리합니다:

- `~/.ivy2/cache`
- `~/.sbt`
- `~/.cache/coursier`

#### 4. 태스크 조합 및 흐름 제어
커스텀 태스크 간의 의존성을 명확하게 설정할 수 있습니다:

```scala
lazy val compileAndPrint = taskKey[Unit]("Compile 후 로그 출력")

compileAndPrint := {
  val _ = (Compile / compile).value
  streams.value.log.info("컴파일 완료!")
}
```

#### 5. 빌드 이벤트 후킹
특정 태스크 실행 전/후의 작업을 정의할 수 있습니다:

```scala
compile := compile
  .dependsOn(myPreCompileTask)
  .value
```

#### 6. 자주 쓰는 설정을 자동화 (AutoPlugin)
자주 쓰는 설정들을 plugin처럼 추출할 수 있습니다:

```scala
// project/MyPlugin.scala
object MyPlugin extends AutoPlugin {
  override def trigger = allRequirements
  override def projectSettings = Seq(
    scalacOptions += "-deprecation"
  )
}
```

---

