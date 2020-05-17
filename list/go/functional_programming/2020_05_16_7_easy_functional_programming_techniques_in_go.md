- 함수형 프로그래밍은 프로그래밍을 함수의 조합으로 생각하고, 상태변화와 변경가능한 데이터를 피하는 것이다.

- 두 가지 중요한 법칙이 있다.

  - 데이터 객체가 만들어진 이후에는 변경되지 않아야 한다.

  - 숨겨진 / 암시적인 상태를 만들지 않는다. 눈에 보이고 명확한 상태만을 취급한다.

- 이 법칙들은 다음을 의미한다.

  - 함수는 함수의 스코프의 바깥에 있는 어떠한 상태도 변화시켜서는 안된다. 즉, 함수 실행은 오직 그 결과로 호출부에 값을 리턴하는 행위만 일어난다. 따라서 외부 상태에는 어떠한 영향도 끼치지 않는다. 이는 이해하기 쉬운 프로그래밍을 가능하게 해준다.

  - 함수의 코드는 멱등성을 갖는다. 함수는 오직 인자로 주어진 인수를 기반으로 한 값만을 반환한다. 전역의 상태를 변화시키거나, 전역의 상태에 의존해서는 안된다. 함수는 항상 동일한 인자에 대해 동일한 결과를 생성한다.

- 함수형 프로그래밍을 한다고 해서, 현재까지 해온 걸 배제한다거나 하는 것은 아니다. 함수형 프로그래밍의 장점을 적절히 사용해서 이득을 취하는 방향으로 선택한다.

## 1급 함수와 고계 함수

- 1급 함수는 변수에 함수를 할당하거나, 함수를 다른 함수에 인수로 전달하거나, 다른 함수에서 함수를 반환할 수 있음을 의미한다.

- 하나 이상의 함수를 매개 변수로 사용하거나, 다른 함수를 결과로 리턴하는 경우 고계 함수라고 한다.

```go
func main() {
	var list = []string{"Orange", "Apple", "Banana", "Grape"}
	// we are passing the array and a function as arguments to mapForEach method.
	var out = mapForEach(list, func(it string) int {
		return len(it)
	})
	fmt.Println(out) // [6, 5, 6, 5]

}

// The higher-order-function takes an array and a function as arguments
func mapForEach(arr []string, fn func(it string) int) []int {
	var newArray = []int{}
	for _, it := range arr {
		// We are executing the method passed
		newArray = append(newArray, fn(it))
	}
	return newArray
}
```

```go
// this is a higher-order-function that returns a function
func add(x int) func(y int) int {
	// A function is returned here as closure
	// variable x is obtained from the outer scope of this method and memorized in the closure
	return func(y int) int {
		return x + y
	}
}

func main() {

	// we are currying the add method to create more variations
	var add10 = add(10)
	var add20 = add(20)
	var add30 = add(30)

	fmt.Println(add10(5)) // 15
	fmt.Println(add20(5)) // 25
	fmt.Println(add30(5)) // 35
}
```

## 순수 함수

- 순수 함수는 전달된 인자를 기반으로 한 값만 반환해야 하며, 전역 상태에 영향을 주거나 의존해서는 안된다.

- 아래는 순수 함수다. 동일한 입력에 대해 항상 동일한 출력을 반환하며, 그 동작이 예측 가능하다.

```go
func sum(a, b int) int {
	return a + b
}
```

- 아래 함수는 외부 상태에 영향을 주는 부작용이 생겨, 동작을 예측하기 어렵다.

```go
var holder = map[string]int{}

func sum(a, b int) int {
	c := a + b
	holder[fmt.Sprintf("%d+%d", a, b)] = c
	return c
}
```

## 재귀

- 함수형 프로그래밍은 반복보다 재귀를 선호한다.

- 반복문 사용

```go
func factorial(num int) int {
	result := 1
	for ; num > 0; num-- {
		result *= num
	}
	return result
}

func main() {
	fmt.Println(factorial(20)) // 2432902008176640000
}
```

- 재귀문 사용

```go
func factorial(num int) int {
	if num == 0 {
		return 1
	}
	return num * factorial(num-1)
}
func main() {
	fmt.Println(factorial(20)) // 2432902008176640000
}
```

- 재귀문의 장점은 코드를 단순하게 하고 가독성을 높혀준다는 것이다.

- 단점은 성능이 반복문보다 대부분의 경우 느릴 수 있다.

- go는 꼬리 재귀 최적화를 지원하지 않는다.

- 가독성과 불변성을 위해 재귀를 사용할 수 있지만, 성능이 중요하거나 반복 횟수가 많을 때는 지양하자.

## 지연 평가(Lazy Evaluation)

- 결과가 필요할 때 까지 해당 연산의 실행을 미루는 것이다.

- 일반적으로 Go는 strict/eager evaluation을 수행한다.

- &&, ||는 lazy evaluation을 수행한다.

- 고계 함수, 클로저, 고루틴, 채널을 사용하여 지연 평가를 사용할 수 있다.

- eager evaluation

```go
func main() {
	fmt.Println(addOrMultiply(true, add(4), multiply(4)))  // 8
	fmt.Println(addOrMultiply(false, add(4), multiply(4))) // 16
}

func add(x int) int {
	fmt.Println("executing add") // this is printed since the functions are evaluated first
	return x + x
}

func multiply(x int) int {
	fmt.Println("executing multiply") // this is printed since the functions are evaluated first
	return x * x
}

func addOrMultiply(add bool, onAdd, onMultiply int) int {
	if add {
		return onAdd
	}
	return onMultiply
}
```

- 아래의 결과를 생성하며 두 함수가 항상 실행된다.

```
executing add
executing multiply
8
executing add
executing multiply
16
```

- 고계 함수를 통해 지연 평가를 수행한다.

```go
func add(x int) int {
	fmt.Println("executing add")
	return x + x
}

func multiply(x int) int {
	fmt.Println("executing multiply")
	return x * x
}

func main() {
	fmt.Println(addOrMultiply(true, add, multiply, 4))
	fmt.Println(addOrMultiply(false, add, multiply, 4))
}

// This is now a higher-order-function hence evaluation of the functions are delayed in if-else
func addOrMultiply(add bool, onAdd, onMultiply func(t int) int, t int) int {
	if add {
		return onAdd(t)
	}
	return onMultiply(t)
}
```

```
executing add
8
executing multiply
16
```

## 참조 투명성(Refrential Transparency)

- 함수형 코드에서는 할당 문이 없다. 즉 프로그램의 변수 값은 한번 정의된 이후에 절대 변경되지 않는다. 따라서 모든 실행 시점에서 변수가 실제 값으로 대체될 수 있다. 따라서 부작용이 발생하지 않는다.

- Go에서 Data Mutation을 엄격하게 제한할 수는 없다.

- 하지만, 순수 함수를 사용하고, 명시적으로 코드에서 데이터의 변형을 피하고, 재할당을 하지 않으면 이를 달성할 수 있다.

- 기본적으로 variables by value로 인자를 전달한다. 슬라이스와 맵은 예외이다. 가능한 참조하는 포인터는 전달하지 않는다.

- 아래의 코드는 참조를 매개 변수로 전달해서 외부 상태를 변경한다.

```go
func main() {
	type Person struct {
		firstName string
		lastName  string
		fullName  string
		age       int
	}
	var getFullName = func(in *Person) string {
		in.fullName = in.firstName + in.lastName // data mutation
		return in.fullName
	}

	john := Person{
		"john", "doe", "", 30,
	}

	fmt.Println(getFullName(&john)) // johndoe
	fmt.Println(john) // {john doe johndoe 30}
}
```

- 매개 변수를 값으로 전달한다. 참조 투명성이 보장된다.

```go
func main() {
	type Person struct {
		firstName string
		lastName  string
		fullName  string
		age       int
	}
	var getFullName = func(in Person) string {
		in.fullName = in.firstName + in.lastName
		return in.fullName
	}

	john := Person{
		"john", "doe", "", 30,
	}

	fmt.Println(getFullName(john))
	fmt.Println(john)
}
```

- 전달된 매개 변수가 맵 또는 슬라이스의 경우에는 이를 적용할 수 없다.


