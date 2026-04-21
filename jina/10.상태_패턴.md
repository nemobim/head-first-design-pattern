# 상태 패턴

## 개념

상태 패턴은 객체 내부 상태가 바뀜에 따라 객체의 행동을 바꿀 수 있는 패턴이다.  
핵심은 "상태별 행동"을 별도 클래스로 캡슐화하고, 행동을 해당 상태 객체에 위임하는 것이다.

`if/else`나 `switch`로 상태를 분기하는 코드는 상태가 늘어날수록 메서드마다 조건문이 폭발한다.  
상태 패턴을 도입하면 각 상태가 자신의 행동을 책임지며, 상태 추가 시 기존 코드를 수정하지 않아도 된다.

## 구조

**State**

상태별 공통 행동을 선언하는 인터페이스 (`State`)

**Concrete State**

각 상태에 해당하는 구체적인 행동을 구현 (`NoQuarterState`, `HasQuarterState`, `SoldState`, `SoldOutState`, `WinnerState`)

**Context**

현재 상태 객체를 보유하고, 행동 요청을 현재 상태에 위임 (`GumballMachine`)

```txt
GumballMachine (Context)
  ├─ state: State          // 현재 상태 보유
  ├─ insertQuarter()       // 상태에 위임
  ├─ ejectQuarter()
  └─ turnCrank()

State (Interface)
  ├─ insertQuarter()
  ├─ ejectQuarter()
  ├─ turnCrank()
  └─ dispense()

NoQuarterState / HasQuarterState / SoldState / SoldOutState / WinnerState (Concrete State)
  └─ 각 상태에서의 행동 구현
```

> "어떻게 처리할지"는 Context가 알고, "지금 어떤 상태인지"는 Concrete State가 결정한다.

## 예시(Class형)

뽑기 기계 구현 (알맹이 5개, 10분의 1 확률로 알맹이 2개 지급)

### 리팩터링 전 (상태를 숫자 상수로 관리)

상태를 숫자 상수로 관리하면 상태가 늘어날수록 모든 메서드에 조건문을 추가해야 한다.

```ts
class GumballMachine {
  SOLD_OUT = 0;
  NO_QUARTER = 1;
  HAS_QUARTER = 2;
  SOLD = 3;

  state = this.SOLD_OUT;
  count = 0;

  insertQuarter(): void {
    if (this.state === this.HAS_QUARTER) {
      console.log("동전은 한 개만 넣어주세요");
    } else if (this.state === this.NO_QUARTER) {
      this.state = this.HAS_QUARTER;
    }
    // ... 상태 추가 시 모든 메서드에 조건문 추가 필요
  }
}
```

### 리팩터링 후 (상태 패턴 적용)

1. State 인터페이스

```ts
interface State {
  insertQuarter(): void;
  ejectQuarter(): void;
  turnCrank(): void;
  dispense(): void;
}
```

2. Context (GumballMachine)

```ts
class GumballMachine {
  soldOutState: State;
  noQuarterState: State;
  hasQuarterState: State;
  soldState: State;
  winnerState: State;

  state: State;
  count: number = 0;

  constructor(numberOfGumballs: number) {
    this.soldOutState = new SoldOutState(this);
    this.noQuarterState = new NoQuarterState(this);
    this.hasQuarterState = new HasQuarterState(this);
    this.soldState = new SoldState(this);
    this.winnerState = new WinnerState(this);

    this.count = numberOfGumballs;
    this.state = numberOfGumballs > 0 ? this.noQuarterState : this.soldOutState;
  }

  insertQuarter(): void {
    this.state.insertQuarter();
  }
  ejectQuarter(): void {
    this.state.ejectQuarter();
  }
  turnCrank(): void {
    this.state.turnCrank();
    this.state.dispense();
  }
  setState(state: State): void {
    this.state = state;
  }
  releaseBall(): void {
    console.log("알맹이를 내보내고 있습니다.");
    if (this.count > 0) this.count--;
  }
}
```

3. Concrete State — NoQuarterState (동전 없음)

```ts
class NoQuarterState implements State {
  gumballMachine: GumballMachine;
  constructor(gumballMachine: GumballMachine) {
    this.gumballMachine = gumballMachine;
  }

  insertQuarter(): void {
    console.log("동전을 넣으셨습니다.");
    this.gumballMachine.setState(this.gumballMachine.hasQuarterState);
  }
  ejectQuarter(): void {
    console.log("동전을 넣어주세요");
  }
  turnCrank(): void {
    console.log("동전을 넣어주세요");
  }
  dispense(): void {
    console.log("동전을 넣어주세요");
  }
}
```

4. Concrete State — HasQuarterState (동전 있음, 10분의 1 확률로 당첨)

```ts
class HasQuarterState implements State {
  randomWinner = () => Math.floor(Math.random() * 10);
  gumballMachine: GumballMachine;
  constructor(gumballMachine: GumballMachine) {
    this.gumballMachine = gumballMachine;
  }

  insertQuarter(): void {
    console.log("동전은 한개만 넣어 주세요.");
  }
  ejectQuarter(): void {
    console.log("동전이 반환됩니다.");
    this.gumballMachine.setState(this.gumballMachine.noQuarterState);
  }
  turnCrank(): void {
    console.log("손잡이를 돌리셨습니다.");
    const winner = this.randomWinner();
    if (winner === 3 && this.gumballMachine.count > 1) {
      this.gumballMachine.setState(this.gumballMachine.winnerState);
    } else {
      this.gumballMachine.setState(this.gumballMachine.soldState);
    }
  }
  dispense(): void {
    console.log("알맹이를 내보낼 수 없습니다.");
  }
}
```

5. Concrete State — WinnerState (당첨: 알맹이 2개 지급)

```ts
class WinnerState implements State {
  gumballMachine: GumballMachine;
  constructor(gumballMachine: GumballMachine) {
    this.gumballMachine = gumballMachine;
  }

  insertQuarter(): void {
    console.log("알맹이를 내보내고 있습니다.");
  }
  ejectQuarter(): void {
    console.log("이미 알맹이를 뽑으셨습니다.");
  }
  turnCrank(): void {
    console.log("손잡이는 한 번만 돌려 주세요.");
  }
  dispense(): void {
    this.gumballMachine.releaseBall();
    if (this.gumballMachine.count === 0) {
      this.gumballMachine.setState(this.gumballMachine.soldOutState);
    } else {
      this.gumballMachine.releaseBall();
      console.log("축하드립니다! 알맹이를 하나 더 받으실 수 있습니다.");
      if (this.gumballMachine.count > 0) {
        this.gumballMachine.setState(this.gumballMachine.noQuarterState);
      } else {
        this.gumballMachine.setState(this.gumballMachine.soldOutState);
      }
    }
  }
}
```

6. 사용

```ts
const gumballMachine = new GumballMachine(5);

gumballMachine.insertQuarter();
gumballMachine.turnCrank();
// → 알맹이 배출, 남은 알맹이 감소

gumballMachine.insertQuarter();
gumballMachine.ejectQuarter();
// → 동전 반환

gumballMachine.insertQuarter();
gumballMachine.turnCrank();
// → 운이 좋으면 WinnerState로 전환되어 알맹이 2개 지급
```

## 상태 패턴 vs 전략 패턴

두 패턴은 클래스 다이어그램이 유사하지만 의도가 다르다.

**상태 패턴**

- 상태 객체에 일련의 행동이 캡슐화된다
- 클라이언트는 어떤 상태인지 몰라도 된다
- 상태 객체 간 전환 로직이 상태 클래스 내부에 있다

**전략 패턴**

- 클라이언트가 Context에게 어떤 전략 객체를 사용할지 직접 지정한다
- 주로 실행 시에 알고리즘을 교체하는 유연성 제공이 목적이다
- 전략 객체는 서로를 모른다

> 상태 패턴: 내부 상태에 따라 스스로 행동이 바뀐다  
> 전략 패턴: 외부에서 행동(알고리즘)을 주입한다

## 언제 사용하면 좋은가?

**✔ 상태에 따라 행동이 완전히 달라질 때**

`insertQuarter()`가 동전 있음/없음/매진 상태마다 완전히 다른 동작을 함

**✔ 상태 분기 조건문이 메서드마다 반복될 때**

모든 메서드에 `if (state === SOLD_OUT) ... else if (state === NO_QUARTER) ...` 패턴이 복잡해질 때

**✔ 상태가 추가될 가능성이 있을 때**

`WinnerState`처럼 새 상태 추가 시 기존 클래스 수정 없이 새 클래스만 추가

**✔ 상태 전환 규칙을 명확하게 관리해야 할 때**

각 상태 클래스가 다음 상태로의 전환을 직접 책임져서 전환 흐름을 추적하기 쉬움

## 프론트엔드 활용 예시

### 비동기 데이터 페칭 (Idle → Loading → Success / Error)

API 호출 상태를 관리하는 가장 흔한 패턴이다.

```ts
interface FetchState<T> {
  render(): React.ReactNode;
}

class IdleState<T> implements FetchState<T> {
  render() {
    return <button onClick={fetch}>불러오기</button>;
  }
}

class LoadingState<T> implements FetchState<T> {
  render() {
    return <Spinner />;
  }
}

class SuccessState<T> implements FetchState<T> {
  constructor(private data: T) {}
  render() {
    return <DataView data={this.data} />;
  }
}

class ErrorState<T> implements FetchState<T> {
  constructor(private message: string) {}
  render() {
    return <ErrorMessage msg={this.message} />;
  }
}
```

React Query / SWR의 `isLoading`, `isError`, `isSuccess` 플래그가 이 구조를 내부적으로 구현한 것이다.

### 폼 제출 흐름 (Editing → Submitting → Success / Error)

```ts
interface FormState {
  canSubmit(): boolean;
  submit(data: FormData): void;
  getButtonLabel(): string;
}

class EditingState implements FormState {
  canSubmit() {
    return true;
  }
  submit(data: FormData) {
    this.form.setState(new SubmittingState(this.form));
    api
      .post(data)
      .then(() => this.form.setState(new SuccessState()))
      .catch((e) => this.form.setState(new ErrorState(e.message)));
  }
  getButtonLabel() {
    return "제출";
  }
}

class SubmittingState implements FormState {
  canSubmit() {
    return false;
  } // 중복 제출 방지
  submit() {}
  getButtonLabel() {
    return "제출 중...";
  }
}

class SuccessState implements FormState {
  canSubmit() {
    return false;
  }
  submit() {}
  getButtonLabel() {
    return "완료!";
  }
}
```

버튼 비활성화, 로딩 스피너, 완료 메시지를 `if`문 없이 상태 객체가 각자 처리한다.

### 미디어 플레이어 (Idle → Playing → Paused → Ended)

```ts
interface PlayerState {
  play(): void;
  pause(): void;
  stop(): void;
  getIcon(): string;
}

class IdleState implements PlayerState {
  play() {
    this.player.setState(new PlayingState(this.player));
  }
  pause() {}
  stop() {}
  getIcon() {
    return "▶";
  }
}

class PlayingState implements PlayerState {
  play() {
    console.log("이미 재생 중입니다.");
  }
  pause() {
    this.player.setState(new PausedState(this.player));
  }
  stop() {
    this.player.setState(new IdleState(this.player));
  }
  getIcon() {
    return "⏸";
  }
}

class PausedState implements PlayerState {
  play() {
    this.player.setState(new PlayingState(this.player));
  }
  pause() {}
  stop() {
    this.player.setState(new IdleState(this.player));
  }
  getIcon() {
    return "▶";
  }
}
```

`getIcon()`을 상태 객체에 맡기기 때문에 UI 컴포넌트는 현재 상태를 직접 확인하지 않아도 된다.

### XState (라이브러리 수준의 상태 패턴)

위 개념을 라이브러리로 구현한 것이 [XState](https://statelyai.com/docs/xstate)다.

```ts
import { createMachine } from "xstate";

const fetchMachine = createMachine({
  initial: "idle",
  states: {
    idle: { on: { FETCH: "loading" } },
    loading: { on: { SUCCESS: "success", ERROR: "error" } },
    success: { type: "final" },
    error: { on: { RETRY: "loading" } },
  },
});
```

React에서 `useMachine(fetchMachine)`으로 바로 사용하며, 상태 전환 규칙을 선언적으로 정의할 수 있다.

### 상태별 정리

| 상황            | 상태 목록                                              |
| --------------- | ------------------------------------------------------ |
| API 페칭        | `idle` → `loading` → `success` / `error`               |
| 폼 제출         | `editing` → `submitting` → `success` / `error`         |
| 미디어 플레이어 | `idle` → `playing` ↔ `paused` → `ended`                |
| 인증 흐름       | `unauthenticated` → `authenticating` → `authenticated` |
| 드래그 앤 드롭  | `idle` → `dragging` → `over` → `dropped`               |

공통점은 상태마다 **허용되는 행동이 다르다**는 것이다. `if`문으로 분기하면 상태가 늘어날수록 복잡해지지만, 상태 패턴을 적용하면 새 상태 클래스만 추가하면 된다.
