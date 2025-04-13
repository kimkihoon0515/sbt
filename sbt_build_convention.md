# 🛠️ build.sbt 작성 컨벤션 (Detailed Convention Guide)

sbt 프로젝트의 일관성과 유지보수성을 높이기 위해 `build.sbt` 작성 시 아래와 같은 컨벤션을 따르는 것을 권장한다.

---

## ✅ 1. 파일 구조 및 정렬 순서

```scala
// 전역 설정 (ThisBuild)
ThisBuild / organization := "com.example"
ThisBuild / scalaVersion := "3.3.5"
ThisBuild / version      := "0.1.0-SNAPSHOT"

// 의존성 import
import Dependencies._

// 프로젝트 설정
lazy val root = (project in file("."))
  .settings(
    name := "my-app",

    // 의존성 설정
    libraryDependencies ++= Seq(
      toolkit,
      toolkitTest % Test
    ),

    // 컴파일 옵션
    scalacOptions ++= Seq(
      "-deprecation",
      "-feature",
      "-unchecked"
    ),

    // 테스트 프레임워크 설정
    testFrameworks += new TestFramework("org.scalatest.tools.Framework")
  )
```

#### 🔹 작성 순서 권장

1. ThisBuild 전역 설정
2. import 문
3. lazy val project 정의
4. settings 내 세부 설정 (name, dependencies, compiler options 등)
5. 태스크 정의 또는 커스텀 설정 (필요 시)

---

## ✅ 2. 코드 스타일 규칙

- `:=`, `+=`, `++=` 등 연산자를 기준으로 **정렬**하여 가독성 확보
- 각 설정 항목 사이에는 **한 줄 공백** 추가
- 리스트 항목은 정렬(알파벳 또는 중요도 순)하여 작성
- `val`, `lazy val`, `.settings(...)`는 그룹으로 정리
- 의존성은 `Seq(...)`로 묶고, 최대 1줄에 1항목 작성

```scala
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % akkaVersion,
  "org.typelevel"     %% "cats-core"  % catsVersion
)
```

---

## ✅ 3. 디렉토리 구조 권장

```
project/
├── build.properties     // sbt 버전 명시
├── Dependencies.scala   // 공통 라이브러리 정의
├── plugins.sbt          // sbt 플러그인 정의

build.sbt                // 프로젝트 설정
src/
├── main/scala/          // 메인 소스
├── test/scala/          // 테스트 코드
```

---

## ✅ 4. 공통 설정 추출 (`commonSettings`)

```scala
lazy val commonSettings = Seq(
  scalacOptions ++= Seq("-deprecation", "-feature"),
  testFrameworks += new TestFramework("org.scalatest.tools.Framework")
)

lazy val core = (project in file("core"))
  .settings(commonSettings: _*)
```

---

## ✅ 5. 의존성 관리

```scala
object Dependencies {
  val catsVersion = "2.10.0"
  val catsCore = "org.typelevel" %% "cats-core" % catsVersion
}
```

```scala
import Dependencies._
libraryDependencies += catsCore
```

---

## ✅ 6. 태스크 정의 컨벤션

```scala
val printHello = taskKey[Unit]("Print Hello to log")

printHello := {
  val log = streams.value.log
  log.info("Hello, SBT!")
}
```

---

## ✅ 7. 플러그인 관리

```scala
val scalafmtVersion = "2.5.0"
addSbtPlugin("org.scalameta" % "sbt-scalafmt" % scalafmtVersion)
```

---

## ✅ 8. 조건부 설정 및 분기 처리

```scala
scalacOptions ++= {
  if (scalaVersion.value.startsWith("3")) Seq("-Xfatal-warnings")
  else Seq("-deprecation", "-unchecked")
}
```

---

## ✅ 9. 주석 및 문서화

```scala
// ▶ 프로젝트 정보 설정
ThisBuild / version := "0.1.0-SNAPSHOT"
```

---

## ✅ 10. 기타 팁

- `.scalafmt.conf`로 코드 스타일 자동화
- `.gitignore`에 `target/`, `.bsp/`, `.idea/` 등 제외
- 멀티 프로젝트에서는 `ThisBuild`로 공통 설정, `lazy val`로 서브 프로젝트 관리

---

## ✅ 11. 고급 실무 적용 팁 및 베스트 프랙티스

### 🔹 프로젝트 구성 확장 전략

- **멀티 프로젝트**를 구성할 땐 `lazy val`과 디렉토리 구조를 잘 나누고, 공통 설정을 `commonSettings`로 분리합니다.
- 예를 들어 다음처럼 구성합니다:

```scala
ThisBuild / organization := "com.example"
ThisBuild / version := "1.0.0"
ThisBuild / scalaVersion := "3.3.5"

lazy val commonSettings = Seq(
  scalacOptions ++= Seq("-deprecation", "-feature"),
  testFrameworks += new TestFramework("org.scalatest.tools.Framework")
)

lazy val core = (project in file("core"))
  .settings(commonSettings: _*)

lazy val api = (project in file("api"))
  .settings(commonSettings: _*)
  .dependsOn(core)

lazy val app = (project in file("app"))
  .settings(commonSettings: _*)
  .dependsOn(core, api)
```

---

### 🔹 스코프 활용 팁

- 설정을 특정 스코프에만 적용하는 방식이 중요합니다:

```scala
Compile / console / scalacOptions --= Seq("-Xfatal-warnings")
Test / fork := true
```

---

### 🔹 sbt 명령 커스터마이징

- 커맨드 스크립트 정의:

```scala
commands += Command.command("greet") { state =>
  println("👋 Hello from custom sbt command!")
  state
}
```

---

### 🔹 CI/CD 환경 최적화

- `.github/workflows/ci.yml` 같은 CI 설정에서 다음 경로를 캐시하여 빌드 속도 향상:

```yaml
- ~/.ivy2/cache
- ~/.sbt
- ~/.cache/coursier
```

- `sbt clean test` 또는 `sbt +test`로 cross build도 병행 가능

---

### 🔹 의존성 충돌 방지 및 exclude

```scala
libraryDependencies += "org.apache.spark" %% "spark-core" % "3.5.0"
  .exclude("org.slf4j", "slf4j-log4j12")
```

- `dependencyTree` 명령어로 충돌 확인:
```
> sbt dependencyTree
```

---

### 🔹 태스크 조합 및 후킹

```scala
lazy val prepare = taskKey[Unit]("Prepare resources")

prepare := {
  val log = streams.value.log
  log.info("Preparing things before compilation!")
}

compile := (compile dependsOn prepare).value
```

---

### 🔹 자주 쓰는 sbt 플러그인 요약

| 플러그인             | 목적                            |
|----------------------|---------------------------------|
| sbt-assembly          | fat JAR 생성                    |
| sbt-native-packager   | Docker/패키징 지원              |
| sbt-revolver          | 빠른 서버 재시작 (`reStart`)     |
| sbt-scalafmt          | 포맷터 적용 (`scalafmtAll`)      |
| sbt-dependency-check  | 보안 취약성 체크                |

---

### 🔹 이름 짓기 네이밍 컨벤션

- 프로젝트 명: kebab-case (e.g. `my-app`)
- 서브 프로젝트 val: `lazy val core = ...`
- 태스크 키: 동사 기반 (`printHello`, `generateDocs`)
- 설정 변수: 명확한 prefix (`akkaVersion`, `playJsonV`)

---

### 🔹 .gitignore 추천 설정

```
target/
.idea/
.bsp/
.metals/
project/target/
project/project/
.history
```

---

이러한 내용을 통해 sbt 설정의 일관성과 재사용성을 확보하고, 협업 및 유지보수 효율을 높일 수 있습니다.
