# 테스트 잘하기

### Q. go에는 왜 assertion 함수가 없나요?

- 분명히 편리한 구석이 있지만, 개발자들이 그러한 것들을 사용해서, 적절한 오류 처리 및 보고에 대해 생각하기를 회피한다는 것을 알게 되었다.
- 적절한 오류 처리라는 것은, 치명적이지 않은(non-fatal) 오류가 발생한 이후에, 중단되지 않고 서버가 계속 동작하는 것이다.
- 적절한 오류 보고는 에러가 발생한 그 지점에서 직접적으로 관찰되어서, 개발자가 긴 스택 트레이스를 해석하지 않아도 된다.
- 명확한 에러는 개발자가 코드에 익숙하지 않을 때 특히 중요하다.
→ 즉, 코드를 작성한 당사자가 아닌 다른 개발자가 디버깅을 할 때 유용하다.

### Q. 테스트를 위해 즐겨 사용하는 헬퍼 함수는 어디있나요?

- go의 testing package는 유닛 테스트를 작성하기 쉽게 되어 있지만, 다른 언어에서 테스팅 프레임워크에서 제공되는 assertion 함수 등은 없다.
- go에 assertion이 없는 이유는, 테스트에 assertion을 제공하지 않는 이유와 일맥상통한다.
- 적절한 에러 처리는, 하나의 테스트가 실패하고 나서 다른 테스트를 실행하는 것이다. 그래서 오류를 디버깅하는 사람이 무엇이 잘못되었는지 완벽한 그림을 파악하는 것이다.
- `isPrime`이라는 함수가 있을 때, 2에 대해서 잘못된 답을 한다고 보고하고 테스트를 끝내는 것보다,
2, 3, 5, 7에 대해서 잘못된 답을 한다고 보고하는 것이 더 유용하다.
- 테스트를 실패하게 코드를 작성한 개발자는 실패한 코드에 대해서 익숙하지 않을 수 있다.
- 좋은 에러 메시지를 작성하는데 투자한 시간은, 테스트가 깨졌을 때 보상받습니다.
- 테스팅 프레임워크는 보통 그들만의 자체 mini-language를 만들고, 조건절, 통제, 메카니즘 출력을 개발하곤 한다.
하지만 go는 이미 이런 능력이 있다. 왜 다시 만들어야 하지?
- 우리는 지금 **Go**로 테스트를 작성하고 있다. Go는 테스트를 직관적이고 이해하기 쉽게 만드는 방법을 배우고, 접근하기 쉬운 언어이다.
- 좋은 오류를 작성하는데 필요한 코드가 너무 반복적이고 귀찮게 느껴지면, 테이블 스타일의 테스트 코드를 작성하는 것이 좋다. go는 데이터 구조 리터럴을 지원하므로, 입력 목록 및 출력 목록을 정의해서 테스트를 더 잘 수행할 수 있다.
- 훌륭한 테스트와 읽기 쉬운 에러 메시지를 작성하는 작업은 많은 테스트 사례에서 보상됩니다. Go의 포준 패키지에서 여러 예시를 볼 수 있습니다.

## 테스트하기 쉬운 Go 코드 작성하기

- 내 코드를 테스트하는 것은 좋은 생각이다.
- 하지만 가끔은 잘못된 것을 테스트하거나, 심지어 아무 것도 테스트하지 못한다!
- 어떻게 테스트해야할지 불분명하기도 하고, 심지어 어떤 것은 테스트할 수 없는 것이라고 생각할 수도 있다.
- 이 글에서는 
코드를 보다 테스트하기 쉽게 만들기위한 트릭과,
코드를 테스트하는데 적용할 수 있는 사고 과정을 다룬다.
- 시작하기 전에, 한 가지 방법을 알고가자. 써드파티 테스팅 프레임워크는 필요 없다.
Go 표준 패키지에 테스트에 필요한 모든 것이 있다. 이 글에서는 오직 이 것들만 사용한다.

- float64 슬라이스의 평균을 구하는 함수가 있다.

```go
func average(values []float64) float64 {
	var total float64= 0
	for _, value := range values {
		total += value
	}
	return total / float64(len(values))
}
```

- 이 것은 완벽하게 테스트하기 좋은 코드이다. 우리는 어떤 입력이든 출력을 알 수 있다. 쉽게 테스트할 수 있다.

```go
func Test_Average(t *testing.T) {
	result := average([]float64{3.5, 2.5, 9.0})
	if result != 5 {
		t.Errorf("for test average, got result %f but expected 5", result)
	}
}
```

- 이 것은 유효하고 유용한 테스트이지만, 하나의 옵션만을 테스트하고 있다. 테스트 케이스 배열을 사용하는 것이 좋다.

- 불행히도 모든 코드가 이렇게 간단하지는 않다. 함수가 다음과 같을 때를 생각해보자.

```go
func averageFromWeb() (float64, error) {
	resp, err := http.DefaultClient.Get("http://our-api.com/numbers")
	if err != nil {
		return 0, err
	}

	values := []float64{}
	if err := json.NewDecoder(resp.Body).Decode(&values); err != nil {
		return 0, err
	}

	var total float64 = 0
	for _, value := range values {
		total += value
	}
	return total / float64(len(values)), nil
}
```

- API에 요청을 보내 값을 가져온다. 여전히 이 코드에 같은 테스트를 할 수 있지만, 매번 같은 값을 위해서 해당 API에 의존한다.

- 그런식으로 테스트를 하더라도 하나의 테스트 케이스로 제한된다.

- API에 대한 커넥션에 의존하고, 테스트할 때 마다 API 콜을 날리는 것은 별로다.

- 여기에서 우리 스스로에게 물어봐야할 것이 있다. 우리가 실제로 테스트하고자 하는 것이 무엇인가?

- go의 http 라이브러리가 http 호출을 할 수 있는지 테스트하는 것인가?

- 그 코드는 작성하지 않았다. 우리가 작성한 모든 코드 즉, 값의 평균읠 계산하는 것을 테스트해야 한다.

- 테스트에서 평균 값을 구하는 함수를 어떻게 호출하고, HTTP 호출은 테스트하지 않아야 하나?

- mock을 사용한다. 그전에 mockable한 코드로 수정해야 한다. 먼저 http 부분을 별도의 함수로 빼야 한다.

```go
func getValues() ([]float64, error) {
	resp, err := http.DefaultClient.Get("http://our-api.com/numbers")
	if err != nil {
		return nil, err
	}

	values := []float64{}
	if err := json.NewDecoder(resp.Body).Decode(&values); err != nil {
		return nil, err
	}
	
	return values, nil
}
```

- 하지만 이 함수는 http 콜을 하므로, API를 호출하지 않고, 선택한 일부 값만 반환하는 mock 코드가 필요하다.

```go
func getValues() ([]float64, error) {
	return []float64{1.0, 2.5}, nil
}
```

- 실제로는 HTTP 호출이 일어나야 하지만, 테스트 시에는 mocking 코드가 동작하기를 원한다.

- 이 함수를 두가지 다른 설정에서 사용할 수 있어야 한다.

- 이 때, 인테퍼이스를 사용하면 편하다. averageFromWeb은 어떤 버전의 getValues가 실행 중인지 알 필요는 없고, 결과만 나오면 된다.

```go
type valueGetter interface {
	GetValues() ([]float64, error)
}
```

```go
func averageFromWeb(valuteGetter ValuteGetter) (float64, error) {
	values, err := valueGetter.GetValues()
	if err != nil {
		return 0, err
	}

	var total float64 = 0
	for _, value := range values {
		total += value
	}
	return total / float64(len(values)), nil
}
```

- 이것은 현재는 괜찮지만, 확장하기 쉽지 않다. 다른 http 서비스 또는 DB를 호출하는 20개의 함수가 있다고 가정해보자.

- 어디를 갈 때마다, 그 모든 서비스와 데이터베이스에 대한 구현을 전달하기는 불편하다. 곧 스파게티 코드가 생길 것이다.

- 이를 우아하게 만들기 위해 구현 세부를 다루는 service 구조체를 만들고 해당 구조체의 함수를 만들 수 있다.

```go
type service struct {
	valueGetter ValuteGetter
}

func (s service) averageFromWeb() (float64, error) {
	values, err := s.valueGetter.GetValues()
	if err != nil {
		return 0, err
	}

	var total float64 = 0
	for _, value := range values {
		total += value
	}
	return total / float64(len(values)), nil
}
```

- 이제 ValueGetter에 대한 구현을 작성한다.

```go
type httpValueGetter struct {}

func (h httpValueGetter) GetValues() ([]float64, error) {
	resp, err := http.DefaultClient.Get("http://our-api.com/numbers")
	if err != nil {
		return nil, err
	}

	values := []float64{}
	if err := json.NewDecoder(resp.Body).Decode(&values); err != nil {
		return nil, err
	}

	return values, nil
}

type mockValueGetter struct {
	values []float64
	err error
}

func (m mockValueGetter) GetValues() ([]float64, error) {
	return m.values, m.err
}
```

- 이제 mockable이 잘 동작하고, 확장가능한 서비스 모델을 함수에 제공할 수 있다. 실제 서비스에서 다음처럼 실제 valueGetter를 초기화할 수 있다.

```go
func main() {
	service := service{valueGetter: httpValueGetter{}}
	
	average, err := service.averageFromWeb()
	if err != nil {
		panic(err)
	}
	
	fmt.Println(average)
}
```

그리고 테스트는 다음과 같은 형태를 갖는다.

```go
func Test_Average(t *testing.T) {
	testError := fmt.Errorf("an example error to compare against")

	tests := []struct{
		name string
		input []float64
		err error
		expectedResult float64
		expectedErr error
	}{
		{
			name: "three normal values",
			input: []float64{3.5, 2.5, 9.0},
			expectedResult: 5,
		},
		{
			name: "handle zeros",
			input: []float64{0, 0},
			expectedResult: 5,
		},
		{
			name: "handle one value",
			input: []float64{15.3},
			expectedResult: 15.3,
		},
		{
			name: "error case",
			input: []float64{3.5, 2.5, 9.0},
			err: testError,
			expectedErr: testError,
		},
	}

	for _, test := range tests{
		service := service{valueGetter: mockValueGetter{
			values: test.input,
			err:    test.err,
		}}

		result, err := service.averageFromWeb()

		if err != test.expectedErr {
			t.Errorf(
				"for average test '%s', got error %v but expected %v",
				test.name,
				err,
				test.expectedErr,
			)
		}

		if result != test.expectedResult {
			t.Errorf(
				"for average test '%s', got result %f but expected %f",
				test.name,
				result,
				test.expectedResult,
			)
		}

	}
}
```

- 이러한 방법론은 http calls, db 쿼리 등 어떤 것도 쉽게 mock 할 수 있게 해준다. 또한 모듈화와 유지보수하기를 쉽게 만들어 준다. 

레퍼런스

- [https://golang.org/doc/faq#assertions](https://golang.org/doc/faq#assertions)
- [https://golang.org/doc/faq#testing_framework](https://golang.org/doc/faq#testing_framework)
- [https://engineering.kablamo.com.au/posts/2020/testing-go](https://engineering.kablamo.com.au/posts/2020/testing-go)