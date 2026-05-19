# 복합 패턴

## 개념

복합 패턴은 **여러 패턴을 함께 사용해서 다양한 디자인 문제를 해결하는 방법**이다.

특정 문제를 해결하기 위해 두 개 이상의 패턴을 조합하여 반복적으로 나타나는 설계 구조를 정의한다.  
각 패턴은 독립적으로도 동작하지만, 함께 사용하면 더 강력하고 유연한 구조를 만들 수 있다.

## 예시: 오리 시뮬레이션 게임

`Quackable` 인터페이스를 중심으로 **어댑터 → 데코레이터 → 추상 팩토리 → 컴포지트 → 옵저버** 순서로 패턴을 점진적으로 추가해나가는 예시다.

### 기본 구조

```typescript
interface Quackable {
  quack(): void;
}

class MallardDuck implements Quackable {
  quack() {
    console.log("꽥꽥");
  }
}

class RedheadDuck implements Quackable {
  quack() {
    console.log("꽥꽥");
  }
}

class DuckCall implements Quackable {
  quack() {
    console.log("꽥꽥");
  }
}

class RubberDuck implements Quackable {
  quack() {
    console.log("삑삑");
  }
}
```

---

## 1단계: 어댑터 패턴 (Adapter Pattern)

`Quackable` 인터페이스를 구현하지 않는 `Goose`를 오리처럼 사용하기 위해 어댑터를 적용한다.

```typescript
class Goose {
  honk() {
    console.log("끽끽");
  }
}

class GooseAdapter implements Quackable {
  goose: Goose;

  constructor(goose: Goose) {
    this.goose = goose;
  }

  quack() {
    this.goose.honk();
  }
}

// 사용
const gooseDuck = new GooseAdapter(new Goose()); // 오리의 탈을 쓴 거위
gooseDuck.quack(); // 끽끽
```

---

## 2단계: 데코레이터 패턴 (Decorator Pattern)

꽥꽥 소리를 낸 횟수를 세는 기능을 `QuackCounter` 데코레이터로 추가한다.  
기존 오리 클래스를 수정하지 않고 행동에 새로운 책임을 덧붙인다.

```typescript
class QuackCounter implements Quackable {
  duck: Quackable;
  static numberOfQuacks: number = 0;

  constructor(duck: Quackable) {
    this.duck = duck;
  }

  quack() {
    this.duck.quack();
    QuackCounter.numberOfQuacks++;
  }

  static getQuacks(): number {
    return QuackCounter.numberOfQuacks;
  }
}

// 사용
const mallard = new QuackCounter(new MallardDuck());
mallard.quack(); // 꽥꽥 (numberOfQuacks: 1)

console.log(`오리 소리 낸 횟수: ${QuackCounter.getQuacks()} 번`);
```

---

## 3단계: 추상 팩토리 패턴 (Abstract Factory Pattern)

오리 객체 생성을 한군데에서 관리하기 위해 추상 팩토리를 도입한다.  
카운터 없는 버전과 카운터 있는 버전을 팩토리로 교체할 수 있다.

```typescript
abstract class AbstractDuckFactory {
  abstract createMallardDuck(): Quackable;
  abstract createRedheadDuck(): Quackable;
  abstract createDuckCall(): Quackable;
  abstract createRubberDuck(): Quackable;
}

// 일반 팩토리
class DuckFactory extends AbstractDuckFactory {
  createMallardDuck() {
    return new MallardDuck();
  }
  createRedheadDuck() {
    return new RedheadDuck();
  }
  createDuckCall() {
    return new DuckCall();
  }
  createRubberDuck() {
    return new RubberDuck();
  }
}

// 카운트 기능이 있는 팩토리
class CountingDuckFactory extends AbstractDuckFactory {
  createMallardDuck() {
    return new QuackCounter(new MallardDuck());
  }
  createRedheadDuck() {
    return new QuackCounter(new RedheadDuck());
  }
  createDuckCall() {
    return new QuackCounter(new DuckCall());
  }
  createRubberDuck() {
    return new QuackCounter(new RubberDuck());
  }
}
```

팩토리를 교체하는 것만으로 시뮬레이터의 동작을 바꿀 수 있다.

```typescript
class DuckSimulator {
  simulate(duckFactory: AbstractDuckFactory) {
    const mallard = duckFactory.createMallardDuck();
    const redhead = duckFactory.createRedheadDuck();
    const duckCall = duckFactory.createDuckCall();
    const rubber = duckFactory.createRubberDuck();
    const goose = new GooseAdapter(new Goose());

    // ...
  }
}
```

---

## 4단계: 컴포지트 패턴 (Composite Pattern)

오리 무리를 개별 오리처럼 다룰 수 있도록 `Flock` 컴포지트를 도입한다.  
개별 객체와 복합 객체를 동일한 `Quackable` 인터페이스로 처리한다.

```typescript
class Flock implements Quackable {
  quackers: Quackable[] = [];

  add(quacker: Quackable) {
    this.quackers.push(quacker);
  }

  quack() {
    for (const quacker of this.quackers) {
      quacker.quack(); // 개별 오리든 Flock이든 동일하게 처리
    }
  }
}
```

Flock 안에 또 다른 Flock을 넣어 트리 구조를 구성할 수 있다.

```typescript
const flockDucks = new Flock();
flockDucks.add(redheadDuck);
flockDucks.add(duckCall);
flockDucks.add(rubberDuck);
flockDucks.add(gooseDuck);

const flockOfMallards = new Flock();
flockOfMallards.add(mallardOne);
flockOfMallards.add(mallardTwo);
flockOfMallards.add(mallardThree);
flockOfMallards.add(mallardFour);

flockDucks.add(flockOfMallards); // Flock 안에 Flock 포함

flockDucks.quack(); // 전체 무리 일제히 꽥꽥
```

---

## 5단계: 옵저버 패턴 (Observer Pattern)

개별 오리의 행동을 외부에서 관찰할 수 있도록 옵저버 패턴을 추가한다.

```typescript
interface QuackObservable {
  registerObserver(observer: Observer): void;
  notifyObservers(): void;
}

interface Observer {
  update(duck: QuackObservable): void;
}

class Observable implements QuackObservable {
  observers: Observer[] = [];

  registerObserver(observer: Observer) {
    this.observers.push(observer);
  }

  notifyObservers() {
    this.observers.forEach((observer) => observer.update(this));
  }
}
```

오리 클래스는 `Observable`을 내부에 위임하여 옵저버 관리 로직을 재사용한다.

```typescript
class MallardDuckObservable implements QuackableObservable {
  observable = new Observable();

  quack() {
    console.log("꽥꽥");
    this.notifyObservers();
  }

  registerObserver(observer: Observer) {
    this.observable.registerObserver(observer);
  }

  notifyObservers() {
    this.observable.notifyObservers();
  }
}

class Quackologist implements Observer {
  update(duck: QuackObservable) {
    console.log("Quackologist: " + duck + " just quacked");
  }
}

// 사용
const mallardDuck = new MallardDuckObservable();
const quackologist = new Quackologist();

mallardDuck.registerObserver(quackologist);
mallardDuck.quack();
// 꽥꽥
// Quackologist: [object Object] just quacked
```

---

## 전체 구조 요약

```txt
Quackable (인터페이스)
  ├─ MallardDuck          ← 기본 오리
  ├─ RedheadDuck          ← 기본 오리
  ├─ DuckCall             ← 기본 오리
  ├─ RubberDuck           ← 기본 오리
  ├─ GooseAdapter         ← 어댑터 패턴: Goose → Quackable
  ├─ QuackCounter         ← 데코레이터 패턴: 꽥꽥 횟수 카운트
  └─ Flock                ← 컴포지트 패턴: 오리 무리를 하나처럼 다룸

AbstractDuckFactory       ← 추상 팩토리 패턴: 오리 생성 책임 분리
  ├─ DuckFactory
  └─ CountingDuckFactory

Observer / Observable     ← 옵저버 패턴: 오리 행동 관찰
```

## 사용된 패턴 정리

| 패턴            | 역할                              |
| --------------- | --------------------------------- |
| **어댑터**      | `Goose`를 `Quackable`로 변환      |
| **데코레이터**  | 기존 오리에 꽥꽥 카운트 기능 추가 |
| **추상 팩토리** | 오리 객체 생성 방식을 캡슐화      |
| **컴포지트**    | 오리 무리를 개별 오리처럼 처리    |
| **옵저버**      | 오리의 행동을 외부에서 감지       |

---

## 모델-뷰-컨트롤러 (MVC)

MVC는 복합 패턴의 대표적인 예시로, **옵저버 · 전략 · 컴포지트** 세 가지 패턴이 결합된 구조다.

### 구성 요소

**Model (모델)**

애플리케이션의 데이터와 비즈니스 로직을 담당한다.  
상태가 변경되면 뷰에게 알림을 보낸다. → **옵저버 패턴**

**View (뷰)**

화면에 표시되는 UI를 담당한다.  
모델의 옵저버 역할을 하며, 모델 상태가 바뀌면 화면을 갱신한다.  
뷰 자체가 중첩된 컴포넌트로 구성될 수 있다. → **컴포지트 패턴**

**Controller (컨트롤러)**

사용자의 입력을 받아 모델을 조작한다.  
뷰는 컨트롤러를 교체함으로써 다른 행동 방식을 취할 수 있다. → **전략 패턴**

### 구조

```txt
사용자 입력
    ↓
Controller   ← 전략 패턴: 뷰가 컨트롤러를 교체하여 행동 변경
    ↓ 조작
  Model       ← 옵저버 패턴: 상태 변경 시 뷰에게 알림
    ↓ 알림
   View        ← 컴포지트 패턴: 중첩된 UI 컴포넌트 구성
```

### 예시: BPM 컨트롤러

비트(BPM)를 관리하는 간단한 MVC 구조를 TypeScript로 표현한다.

```typescript
// Model
interface BeatModelInterface {
  getBPM(): number;
  setBPM(bpm: number): void;
  registerObserver(observer: BeatObserver): void;
  removeObserver(observer: BeatObserver): void;
}

interface BeatObserver {
  updateBeat(): void;
}

class BeatModel implements BeatModelInterface {
  private bpm: number = 90;
  private observers: BeatObserver[] = [];

  getBPM() {
    return this.bpm;
  }

  setBPM(bpm: number) {
    this.bpm = bpm;
    this.notifyObservers();
  }

  registerObserver(observer: BeatObserver) {
    this.observers.push(observer);
  }

  removeObserver(observer: BeatObserver) {
    this.observers = this.observers.filter((o) => o !== observer);
  }

  private notifyObservers() {
    this.observers.forEach((o) => o.updateBeat());
  }
}
```

```typescript
// View
class BeatView implements BeatObserver {
  private model: BeatModelInterface;

  constructor(model: BeatModelInterface) {
    this.model = model;
    model.registerObserver(this);
  }

  updateBeat() {
    console.log(`현재 BPM: ${this.model.getBPM()}`);
  }
}
```

```typescript
// Controller
class BeatController {
  private model: BeatModelInterface;

  constructor(model: BeatModelInterface) {
    this.model = model;
  }

  increaseBPM() {
    this.model.setBPM(this.model.getBPM() + 1);
  }

  decreaseBPM() {
    this.model.setBPM(this.model.getBPM() - 1);
  }

  setBPM(bpm: number) {
    this.model.setBPM(bpm);
  }
}
```

```typescript
// 조립 및 실행
const model = new BeatModel();
const view = new BeatView(model); // 뷰가 모델을 구독
const controller = new BeatController(model);

controller.setBPM(120); // 현재 BPM: 120
controller.increaseBPM(); // 현재 BPM: 121
controller.decreaseBPM(); // 현재 BPM: 120
```

### MVC에서 각 패턴의 역할

| 패턴         | MVC에서의 적용                               |
| ------------ | -------------------------------------------- |
| **옵저버**   | 모델이 뷰에게 상태 변경을 알림               |
| **전략**     | 뷰에 연결된 컨트롤러를 교체해 다른 동작 부여 |
| **컴포지트** | 뷰 내부의 UI 컴포넌트를 트리 구조로 구성     |

---

## 언제 사용하면 좋은가?

**✔ 단일 패턴으로는 해결하기 어려운 복잡한 시스템을 설계할 때**

여러 책임이 얽혀 있는 도메인에서 각 패턴이 맡은 영역을 분리해 복잡도를 낮춘다.

**✔ 기존 코드를 변경하지 않고 기능을 점진적으로 확장해야 할 때**

데코레이터, 어댑터, 팩토리를 조합하면 원본 클래스를 건드리지 않고 기능을 추가할 수 있다.

**✔ 개별 객체와 복합 객체를 동일하게 처리해야 할 때**

컴포지트 패턴을 중심으로 옵저버나 데코레이터를 결합하면 통일된 인터페이스를 유지하면서 기능을 확장할 수 있다.
