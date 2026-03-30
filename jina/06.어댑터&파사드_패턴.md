# 어댑터 패턴 & 파사드 패턴

## 어댑터 패턴 (Adapter Pattern)

### 정의

어댑터 패턴은 특정 인터페이스를 클라이언트에서 요구하는 다른 인터페이스로 변환하는 패턴. 인터페이스가 호환되지 않아 같이 쓸 수 없었던 클래스의 사용을 가능하게 함.

### 동작 원리

1. 클라이언트에서 타깃 인터페이스로 메소드를 호출해 어댑터에 요청을 보냄
2. 어댑터는 어댑티 인터페이스로 그 요청을 어댑티에 관한 메소드 호출로 변환
3. 클라이언트는 호출 결과를 받지만 중간에 어댑터가 있음을 모름

### 구조

```
클라이언트 → request()
     ↓
어댑터 → translatedRequest()
     ↓
어댑티 (모든 요청은 어댑티에 위임)
```

### 어댑터 종류

#### 1. 객체 어댑터

- 구성방식: 합성을 사용
- 어댑티를 포함하는 방식

#### 2. 클래스 어댑터

- 구성방식: 다중상속을 사용
- Target 인터페이스를 구현하면서 동시에 Adaptee를 상속받는 방식

### 예제 1: 오리와 칠면조

#### 문제 상황

Duck 객체가 모자라서 Turkey 객체를 사용해야 하는 상황

#### 인터페이스 정의

```typescript
interface Duck {
	quack(): void;
	fly(): void;
}

interface Turkey {
	gobble(): void;
	fly(): void;
}
```

#### 구현 클래스

```typescript
class MallardDuck implements Duck {
	quack(): void {
		console.log('꽥꽥');
	}
	fly(): void {
		console.log('날고 있어요!!');
	}
}

class WildTurkey implements Turkey {
	gobble(): void {
		console.log('골골');
	}
	fly(): void {
		console.log('짧은 거리를 날고 있어요!');
	}
}
```

#### 어댑터 구현

```typescript
class TurkeyAdapter implements Duck {
	turkey: Turkey;

	constructor(turkey: Turkey) {
		this.turkey = turkey;
	}

	quack(): void {
		this.turkey.gobble();
	}

	fly(): void {
		for (let i = 0; i < 5; i++) {
			this.turkey.fly();
		}
	}
}
```

#### 사용 예제

```typescript
const duck = new MallardDuck();
const turkey = new WildTurkey();
const turkeyAdapter = new TurkeyAdapter(turkey);

const testDuck = (duck: Duck) => {
	duck.quack();
	duck.fly();
};

console.log('\n칠면조가 말하길');
turkey.gobble();
turkey.fly();
// 출력: 골골
//      짧은 거리를 날고 있어요!

console.log('\n오리가 말하길');
testDuck(duck);
// 출력: 꽥꽥
//      날고 있어요!!

console.log('\n칠면조 어댑터가 말하길');
testDuck(turkeyAdapter);
// 출력: 골골
//      짧은 거리를 날고 있어요! (5번 반복)
```

### 예제 2: Enumeration을 Iterator에 적응시키기

```typescript
interface Iterator {
	hasNext(): boolean;
	next(): any;
	remove(): void;
}

interface Enumeration {
	hasMoreElements(): boolean;
	nextElement(): any;
}

class EnumerationAdapter implements Iterator {
	enumeration: Enumeration;

	constructor(enumeration: Enumeration) {
		this.enumeration = enumeration;
	}

	hasNext(): boolean {
		return this.enumeration.hasMoreElements();
	}

	next(): any {
		return this.enumeration.nextElement();
	}

	remove(): void {
		// Enumeration에는 remove() 메소드가 없기 때문에 예외 발생
		throw new Error('Method not implemented.');
	}
}
```

---

## 파사드 패턴 (Facade Pattern)

### 정의

파사드 패턴은 서브 시스템에 있는 일련의 인터페이스를 통합 인터페이스로 묶어주는 패턴. 또한 고수준 인터페이스도 정의하므로 서브시스템을 더 편리하게 사용 가능.

### 목적

쓰기 쉬운 인터페이스를 제공하는 파사드 클래스 구현을 통한 복잡한 시스템의 편리한 사용.

### 예제: 홈 시어터 시스템

```typescript
class HomeTheaterFacade {
	amplifier: Amplifier;
	turner: Turner;
	player: Player;
	projector: Projector;
	lights: Lights;
	screen: Screen;
	popcorn: PopcornPopper;

	constructor(
		amplifier: Amplifier,
		turner: Turner,
		player: Player,
		projector: Projector,
		lights: Lights,
		screen: Screen,
		popcorn: PopcornPopper,
	) {
		this.amplifier = amplifier;
		this.turner = turner;
		this.player = player;
		this.projector = projector;
		this.lights = lights;
		this.screen = screen;
		this.popcorn = popcorn;
	}

	watchMovie(movie: string) {
		this.lights.dim(10);
		this.turner.on();
		this.turner.setInput(this.turner.TunerInput.DVD);
		this.amplifier.on();
		this.amplifier.setVolume(5);
		this.amplifier.setInput(this.amplifier.Input.DVD);
		this.projector.on();
		this.projector.setInput(this.projector.Input.DVD);
		this.screen.down();
		this.popcorn.on();
		this.popcorn.pop();
	}

	endMovie() {
		this.lights.on();
		this.turner.off();
		this.turner.setInput(this.turner.TunerInput.DVD);
		this.amplifier.off();
		this.amplifier.setVolume(0);
		this.amplifier.setInput(this.amplifier.Input.DVD);
		this.projector.off();
		this.projector.setInput(this.projector.Input.DVD);
	}
}
```

#### 사용 예제

```typescript
const amplifier = new Amplifier();
const turner = new Turner();
const player = new Player();
const projector = new Projector();
const lights = new Lights();
const screen = new Screen();
const popcorn = new PopcornPopper();

const homeTheater = new HomeTheaterFacade(
	amplifier,
	turner,
	player,
	projector,
	lights,
	screen,
	popcorn,
);

homeTheater.watchMovie('The Dark Knight');
homeTheater.endMovie();
```

---

## 최소 지식 원칙 (Principle of Least Knowledge)

### 원칙

객체 사이의 상호작용은 될 수 있으면 아주 가까운 '친구' 사이에만 허용.

### 규칙

메소드 호출은 다음 4가지 경우에만 허용:

```typescript
class MyClass {
	method() {
		// 1. 객체 자신의 메소드
		this.anotherMethod();

		// 2. 메소드에 매개변수로 전달된 객체의 메소드
		function doSomething(obj: SomeClass) {
			obj.someMethod();
		}

		// 3. 그 메소드에서 생성하거나 인스턴스를 만든 객체의 메소드
		const newObj = new AnotherClass();
		newObj.someMethod();

		// 4. 그 객체에 속하는 구성 요소(인스턴스 변수)의 메소드
		this.component.someMethod();
	}
}
```

### 나쁜 예

여러 객체를 거쳐서 메소드를 호출하는 경우

```typescript
class BadExample {
	getTemp(): number {
		// station에서 thermometer를 받아오고
		// 다시 thermometer에서 temperature를 받아옴
		// 너무 많은 객체와 결합됨!
		return this.station.getThermometer().getTemperature();
	}
}
```

### 좋은 예

직접 요청하여 내부 구조를 알 필요가 없게 함

```typescript
class GoodExample {
	getTemp(): number {
		// station에게 직접 요청
		// station 내부 구조를 알 필요 없음
		return this.station.getTemperature();
	}
}

// Station 클래스 내부에서 처리
class Station {
	thermometer: Thermometer;

	getTemperature(): number {
		// 외부에 thermometer를 노출하지 않음
		return this.thermometer.getTemperature();
	}
}
```

### 파사드 패턴과의 관계

HomeTheaterFacade 클래스가 바로 최소 지식 원칙을 잘 적용한 예. 클라이언트 코드는 복잡한 내부 시스템을 알 필요없이 파사드를 통해서만 상호작용.

### 장점

- **결합도 감소**: 시스템 간의 의존성이 줄어듦
- **유지보수 용이**: 한 부분의 변경이 다른 부분에 영향을 덜 미침
- **이해하기 쉬움**: 각 객체가 간단한 인터페이스만 제공

### 단점

- 래퍼 클래스를 더 만들어야 할 수 있어 복잡성 증가
- 실행 시간이 느려질 수 있음

---

## JavaScript 실무 예시

### 어댑터 패턴 실무 예시

#### 1. 외부 라이브러리 API 통합

서로 다른 결제 API를 통일된 인터페이스로 사용

```javascript
// 기존 PayPal API
class PayPalAPI {
	sendPayment(amount, email) {
		console.log(`PayPal: ${amount}원을 ${email}로 송금`);
		return { success: true, transactionId: 'PP-' + Date.now() };
	}
}

// 기존 Stripe API
class StripeAPI {
	charge(cents, accountId) {
		console.log(`Stripe: ${cents}센트를 ${accountId}에 청구`);
		return { status: 'completed', id: 'ST-' + Date.now() };
	}
}

// 통합 결제 인터페이스
class PaymentProcessor {
	processPayment(amount, account) {
		throw new Error('구현 필요');
	}
}

// PayPal 어댑터
class PayPalAdapter extends PaymentProcessor {
	constructor() {
		super();
		this.paypal = new PayPalAPI();
	}

	processPayment(amount, account) {
		const result = this.paypal.sendPayment(amount, account);
		return {
			success: result.success,
			transactionId: result.transactionId,
			provider: 'PayPal',
		};
	}
}

// Stripe 어댑터
class StripeAdapter extends PaymentProcessor {
	constructor() {
		super();
		this.stripe = new StripeAPI();
	}

	processPayment(amount, account) {
		const cents = amount * 100; // 원을 센트로 변환
		const result = this.stripe.charge(cents, account);
		return {
			success: result.status === 'completed',
			transactionId: result.id,
			provider: 'Stripe',
		};
	}
}

// 사용 예시
const paymentMethods = [new PayPalAdapter(), new StripeAdapter()];

paymentMethods.forEach((payment) => {
	const result = payment.processPayment(10000, 'user@example.com');
	console.log(result);
});
```

#### 2. 레거시 코드와 신규 코드 통합

```javascript
// 레거시 API (콜백 기반)
class LegacyAPI {
	fetchData(callback) {
		setTimeout(() => {
			callback(null, { data: '레거시 데이터' });
		}, 1000);
	}
}

// 신규 API (Promise 기반)
class ModernAPI {
	async getData() {
		return new Promise((resolve) => {
			setTimeout(() => {
				resolve({ data: '모던 데이터' });
			}, 1000);
		});
	}
}

// 레거시를 Promise로 변환하는 어댑터
class LegacyToPromiseAdapter {
	constructor() {
		this.legacy = new LegacyAPI();
	}

	async getData() {
		return new Promise((resolve, reject) => {
			this.legacy.fetchData((error, data) => {
				if (error) reject(error);
				else resolve(data);
			});
		});
	}
}

// 통일된 방식으로 사용
async function fetchAllData() {
	const apis = [new LegacyToPromiseAdapter(), new ModernAPI()];

	const results = await Promise.all(apis.map((api) => api.getData()));

	console.log(results);
}
```

#### 3. 로컬 스토리지 어댑터

```javascript
// 다양한 저장소를 통일된 인터페이스로 사용
class StorageAdapter {
	get(key) {}
	set(key, value) {}
	remove(key) {}
}

// LocalStorage 어댑터
class LocalStorageAdapter extends StorageAdapter {
	get(key) {
		const value = localStorage.getItem(key);
		return value ? JSON.parse(value) : null;
	}

	set(key, value) {
		localStorage.setItem(key, JSON.stringify(value));
	}

	remove(key) {
		localStorage.removeItem(key);
	}
}

// SessionStorage 어댑터
class SessionStorageAdapter extends StorageAdapter {
	get(key) {
		const value = sessionStorage.getItem(key);
		return value ? JSON.parse(value) : null;
	}

	set(key, value) {
		sessionStorage.setItem(key, JSON.stringify(value));
	}

	remove(key) {
		sessionStorage.removeItem(key);
	}
}

// IndexedDB 어댑터 (간소화)
class IndexedDBAdapter extends StorageAdapter {
	constructor(dbName = 'myDB') {
		super();
		this.dbName = dbName;
		this.cache = new Map();
	}

	async get(key) {
		return this.cache.get(key) || null;
	}

	async set(key, value) {
		this.cache.set(key, value);
	}

	async remove(key) {
		this.cache.delete(key);
	}
}

// 사용 예시
class UserPreferences {
	constructor(storage) {
		this.storage = storage;
	}

	saveTheme(theme) {
		this.storage.set('theme', theme);
	}

	getTheme() {
		return this.storage.get('theme') || 'light';
	}
}

// 필요에 따라 저장소 교체 가능
const prefs = new UserPreferences(new LocalStorageAdapter());
prefs.saveTheme('dark');
```

### 파사드 패턴 실무 예시

#### 1. 복잡한 DOM 조작 라이브러리

```javascript
// 복잡한 DOM, 이벤트, 애니메이션을 간단한 인터페이스로 제공
class DOMFacade {
	constructor(selector) {
		this.element = document.querySelector(selector);
	}

	// 여러 작업을 하나로 통합
	show(duration = 300) {
		this.element.style.display = 'block';
		this.element.style.opacity = '0';
		this.fadeIn(duration);
	}

	hide(duration = 300) {
		this.fadeOut(duration, () => {
			this.element.style.display = 'none';
		});
	}

	fadeIn(duration) {
		const start = performance.now();
		const animate = (currentTime) => {
			const elapsed = currentTime - start;
			const progress = Math.min(elapsed / duration, 1);
			this.element.style.opacity = progress;

			if (progress < 1) {
				requestAnimationFrame(animate);
			}
		};
		requestAnimationFrame(animate);
	}

	fadeOut(duration, callback) {
		const start = performance.now();
		const animate = (currentTime) => {
			const elapsed = currentTime - start;
			const progress = Math.min(elapsed / duration, 1);
			this.element.style.opacity = 1 - progress;

			if (progress < 1) {
				requestAnimationFrame(animate);
			} else if (callback) {
				callback();
			}
		};
		requestAnimationFrame(animate);
	}

	on(event, handler) {
		this.element.addEventListener(event, handler);
		return this; // 체이닝 가능
	}

	addClass(className) {
		this.element.classList.add(className);
		return this;
	}

	removeClass(className) {
		this.element.classList.remove(className);
		return this;
	}
}

// 사용 예시
const modal = new DOMFacade('#modal');
modal
	.addClass('active')
	.show(500)
	.on('click', () => modal.hide(500));
```

#### 2. HTTP 요청 파사드

```javascript
// fetch, 에러 핸들링, 인증을 하나로 통합
class APIFacade {
	constructor(baseURL) {
		this.baseURL = baseURL;
		this.token = null;
	}

	setToken(token) {
		this.token = token;
	}

	async request(endpoint, options = {}) {
		const url = `${this.baseURL}${endpoint}`;
		const headers = {
			'Content-Type': 'application/json',
			...options.headers,
		};

		if (this.token) {
			headers['Authorization'] = `Bearer ${this.token}`;
		}

		try {
			const response = await fetch(url, {
				...options,
				headers,
			});

			if (!response.ok) {
				throw new Error(`HTTP ${response.status}: ${response.statusText}`);
			}

			return await response.json();
		} catch (error) {
			console.error('API 요청 실패:', error);
			throw error;
		}
	}

	async get(endpoint) {
		return this.request(endpoint, { method: 'GET' });
	}

	async post(endpoint, data) {
		return this.request(endpoint, {
			method: 'POST',
			body: JSON.stringify(data),
		});
	}

	async put(endpoint, data) {
		return this.request(endpoint, {
			method: 'PUT',
			body: JSON.stringify(data),
		});
	}

	async delete(endpoint) {
		return this.request(endpoint, { method: 'DELETE' });
	}
}

// 사용 예시
const api = new APIFacade('https://api.example.com');
api.setToken('your-token-here');

async function loadUser() {
	try {
		const user = await api.get('/users/me');
		console.log(user);
	} catch (error) {
		console.error('사용자 로드 실패:', error);
	}
}
```

#### 3. 폼 검증 파사드

```javascript
// 복잡한 폼 검증 로직을 간단한 인터페이스로 제공
class FormValidationFacade {
	constructor(formId) {
		this.form = document.getElementById(formId);
		this.errors = {};
		this.validators = {
			required: (value) => value.trim() !== '',
			email: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
			minLength: (value, min) => value.length >= min,
			maxLength: (value, max) => value.length <= max,
			number: (value) => !isNaN(value) && value.trim() !== '',
		};
	}

	validate(rules) {
		this.errors = {};
		let isValid = true;

		for (const [fieldName, fieldRules] of Object.entries(rules)) {
			const input = this.form.elements[fieldName];
			if (!input) continue;

			const value = input.value;

			for (const rule of fieldRules) {
				const { type, message, ...params } = rule;
				const validator = this.validators[type];

				if (validator && !validator(value, ...Object.values(params))) {
					this.errors[fieldName] = message;
					this.showError(input, message);
					isValid = false;
					break;
				} else {
					this.clearError(input);
				}
			}
		}

		return isValid;
	}

	showError(input, message) {
		input.classList.add('error');
		let errorEl = input.nextElementSibling;

		if (!errorEl || !errorEl.classList.contains('error-message')) {
			errorEl = document.createElement('span');
			errorEl.className = 'error-message';
			input.parentNode.insertBefore(errorEl, input.nextSibling);
		}

		errorEl.textContent = message;
	}

	clearError(input) {
		input.classList.remove('error');
		const errorEl = input.nextElementSibling;
		if (errorEl && errorEl.classList.contains('error-message')) {
			errorEl.remove();
		}
	}

	getErrors() {
		return this.errors;
	}
}

// 사용 예시
const formValidator = new FormValidationFacade('signup-form');

document.getElementById('signup-form').addEventListener('submit', (e) => {
	e.preventDefault();

	const isValid = formValidator.validate({
		username: [
			{ type: 'required', message: '사용자명은 필수입니다' },
			{ type: 'minLength', min: 3, message: '최소 3자 이상이어야 합니다' },
		],
		email: [
			{ type: 'required', message: '이메일은 필수입니다' },
			{ type: 'email', message: '올바른 이메일 형식이 아닙니다' },
		],
		age: [
			{ type: 'required', message: '나이는 필수입니다' },
			{ type: 'number', message: '숫자만 입력 가능합니다' },
		],
	});

	if (isValid) {
		console.log('폼 제출 성공!');
	} else {
		console.log('검증 실패:', formValidator.getErrors());
	}
});
```

#### 4. 차트 라이브러리 파사드

```javascript
// Chart.js, D3.js 등 복잡한 차트 라이브러리를 간단하게 사용
class ChartFacade {
	constructor(canvasId) {
		this.canvas = document.getElementById(canvasId);
		this.ctx = this.canvas.getContext('2d');
	}

	drawLineChart(data, options = {}) {
		// 복잡한 Chart.js 설정을 내부에서 처리
		const config = {
			type: 'line',
			data: {
				labels: data.labels,
				datasets: [
					{
						label: options.title || '데이터',
						data: data.values,
						borderColor: options.color || 'rgb(75, 192, 192)',
						tension: 0.1,
					},
				],
			},
			options: {
				responsive: true,
				plugins: {
					legend: { display: options.showLegend !== false },
					title: { display: true, text: options.title || '' },
				},
			},
		};

		// Chart.js 라이브러리 사용 (실제로는 new Chart() 호출)
		console.log('라인 차트 생성:', config);
	}

	drawBarChart(data, options = {}) {
		console.log('바 차트 생성');
	}

	drawPieChart(data, options = {}) {
		console.log('파이 차트 생성');
	}

	updateChart(newData) {
		console.log('차트 업데이트');
	}

	destroy() {
		console.log('차트 제거');
	}
}

// 사용 예시
const chart = new ChartFacade('myChart');
chart.drawLineChart(
	{
		labels: ['1월', '2월', '3월', '4월', '5월'],
		values: [12, 19, 3, 5, 2],
	},
	{
		title: '월별 매출',
		color: 'rgb(255, 99, 132)',
		showLegend: true,
	},
);
```

### 실무 적용

#### 어댑터 패턴 사용 시기

- 외부 라이브러리나 API를 통합할 때
- 레거시 코드를 리팩토링할 때
- 여러 벤더의 서비스를 통일된 인터페이스로 사용할 때
- 테스트를 위한 목(Mock) 객체를 만들 때

#### 파사드 패턴 사용 시기

- 복잡한 시스템을 단순화해야 할 때
- 여러 단계의 초기화나 설정이 필요할 때
- 라이브러리나 프레임워크를 래핑할 때
- 마이크로서비스 간 통신을 추상화할 때
