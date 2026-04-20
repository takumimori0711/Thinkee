# サンプル図

## シーケンス図

```kroki
plantuml

@startuml
participant User
participant System

User -> System: リクエスト
System --> User: レスポンス
@enduml
```

## クラス図

```kroki
plantuml

@startuml
class Animal {
  +name: String
  +speak(): void
}
class Dog extends Animal {
  +speak(): void
}
@enduml
```
