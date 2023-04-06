# Interface

Go Interface

- Concept
- Type method
- Receiver (Value / Pointer)
- Type Assertion
- Interface Pattern
  - Empty Interface
  - Embedding Interface


<br>
<br>

### Interface

1. 인터페이스(타입 메서드) 선언
    1. 함수 인자로 인터페이스를 받고, 인터페이스 타입 메서드를 활용하여 구현
2. 인스턴스 및 메서드 구현
3. 인스턴스 객체 생성 후 구현한 함수의 인자로 호출

```go
// 1. 인터페이스 (타입 메서드) 선언
type Develop interface {
  Docs(time.Time) bool
 // Docs()     메서드명은 고유해야 함.  인자 & 리턴 타입 값이 달라도 불가능
  Coding(code interface{}) (int,error)
	Debuging(line ...int)
	Test(dummies interface{}) (bool)
}
```

- 인터페이스 선언 예시
    
    [ Todo-List ]
    
    ```jsx
    type Todos interface {
    	FindAll() ([]domain.Todo, error)
    	FindByTodoId(todoId string) (domain.Todo, error)
    	FindDeletedByTodoId(todoId string) (domain.Todo, error)
    
    	Create(domain.Todo) (domain.Todo, error)
    	Update(string, domain.Todo) (domain.Todo, error)
    	Delete(domain.Todo) error
    }
    ```
    
    [ kubernetes ]
    
    ```jsx
    // Abstract interface to disk operations.
    type diskManager interface {
    	MakeGlobalPDName(disk fcDisk) string
    	MakeGlobalVDPDName(disk fcDisk) string
    	// Attaches the disk to the kubelet's host machine.
    	AttachDisk(b fcDiskMounter) (string, error)
    	// Detaches the disk from the kubelet's host machine.
    	DetachDisk(disk fcDiskUnmounter, devicePath string) error
    	// Detaches the block disk from the kubelet's host machine.
    	DetachBlockFCDisk(disk fcDiskUnmapper, mntPath, devicePath string) error
    }
    
    // https://github.com/kubernetes/kubernetes/blob/master/pkg/volume/fc/disk_manager.go
    ```
    

```go
// (a). 함수 인자로 인터페이스를 받고, 인터페이스 메서드를 활용하여 구현
func Project(**dev Develop**,ops Operate,...) bool {
	for Success {
	  dev.Coding(code interface{}) (int,error)
		dev.Debuging(line ...int)
		dev.Test(dummies interface{}) (bool)
		
		ops.Deploy(..)..
		ops.Monitoring(..).. 	
	}
}
```

- 인터페이스 활용 예시
    
    [ Todo-List ].  생성자 선언
    
    ```jsx
    func NewTodoService(todoRepository **repository.Todos**) *TodoService {
    	return &TodoService{
    		todoRepository: todoRepository,
    	}
    }
    ```
    

```go
인스턴스의 조건 & 생성 및 할당 방법
- interface 에 정의된 모든 타입 메서드를 포함 (= 구현)
- 일반적으로 인스턴스는 구조체를 의미

// 3.인스턴스 및 타입 메서드 구현
type Backend struct {
  GroupName string
  Member int
  Language string
	Details interface{}
}
// 인터페이스에 정의된 모든 메서드 구현 
func (b *Backend) Coding(code interface{}) (int,error) {
	for l, c := range code {
  ...
  }
	return l,err
}

func (b Backend) Debugging(line ...int){
	...
}
func (b Backend) Test(dummies interface{}) bool {
	res := fmt.Sprintln("Method3 is ", i.Details, s)
	return res
}
```

```go
func main() {
	var infra_dev Develop
	infra_dev = &Backend{"Infra",8,"Golang"}   // 인스턴스 할당
	infra_dev.Coding(code)
	if !infra_dev.Test(dummies) {
		infra_dev.Debuggin(line)
	}

// 4. 인스턴스 생성 후 구현한 함수의 인자로 호출
  res := Project(infra_dev, ops, ...)
  fmt.Println(res)
}
```

Value 리시버  vs. Pointer 리시버

```go
func (b *Backend) Coding(code interface{}) (int,error) { } 
func (b Backend) Debugging(line ...int){ }
func (b Backend) Test(dummies interface{}) bool { }

// var dev_group Develop = Backend{ .. } X

var dev_group Develop = &Backend{ .. }  O
```
<br>
<br>

### Type assertion

        **런타임 에러 (panic)를 피하기 위해 검증하는 과정**

인터페이스는 모든 유형의 타입을 지원하기 때문에, 
`var Emp_Name interface{}`  변수가 `“12303510” or 12303510` 와 같은 인터페이스 값을 사용하려면 타입을 분명히 해야 할 필요가 있음.

(  **x.(T) 형태로 assertion 이 이루어지며  x 는 interface  ,  T 는 Type 값을 의미 )**

```go
var infName interface{} = "12303510"

var randInt int = infName    // 컴파일 에러 발생  -  Require Type assertion
// cannot use infName (variable of type interface{}) as int value in variable declaration: **need type assertion**

var randInt int = infName.(int)  // 런타임 에러 (panic) 발생
// panic: interface conversion: interface {} is string, not int

// Type assertion
randInt, ok := infName.(int)   // ok 를 통해 변환 가능여부를 검증하여 randInt에 의도하지 않은 값 할당을 방지
// 0, false

if !ok {
  randInt = infName.(string)   
}

/* 단일 타입을 가지는 인터페이스이면 위와 같이 **!ok** 처럼 쉽게 처리할 수 있지만, 
   여러 타입을 가지는 인터페이스이면 ***switch*** 문을 통해 처리하는 방법도 있음 */
switch infName.(type) {     // .(type) 코드는 switch 절에서만 사용 가능
	case Custom:
		fmt.Println("CUSTOM")
    ...
	case string:
		fmt.Println("STRING")
    ...
	case interface{}:  // interface{} 대신 default 로 대체 가능
		fmt.Println("OTHERS")
    ...
	}

// Log 가 Logger 를 인터페이스로 가지는지 확인하는 type Assertion

var l Log
_, ok := l.(Logger)

// invalid type assertion: l.(Logger) (non-interface type Log on left)

var l Log
_, ok := interface{}(l).(Logger)

```

<aside>
💡 **덕 타이핑**

일단 객체를 정의한 후  해당 객체가 인터페이스에 정의한 메서드를 모두 포함하는 경우
인스턴스를 인터페이스로 활용이 가능

</aside>

## Interface Pattern

**Empty Interface**

<aside>
💡 빈 인터페이스   ( empty Interface )

interface{ } 처럼 빈 인터페이스를 선언 할 수 있고, 정의된 메서드가 없으므로 모든 타입에 적용이 가능

런타임에서 동적으로 활용하기 위해 사용함.    (Java - Object , C++ - void*)

Go 1.18 버전 이후 `any` 로 사용 가능    ( any 는 interface{ ] 의 alias )

</aside>

```go
type Body struct {
    Msg interface{}
}

func main() {

    b := Body{"Hello there"}
    fmt.Printf("%#v %T\n", b.Msg, b.Msg)

    b.Msg = 5
    fmt.Printf("%#v %T\n", b.Msg, b.Msg)
}

// json 파싱 시 struct 구조를 알기 어려울 때
test := `{
    "database": {
        "driver": "kinxsql",
        "version": 2.8.0
    },
    "app": "kinx.go"
}`
data := make(map[string]interface{})
err := json.Unmarshal([]byte(test), &data)

if err != nil {
    fmt.Println(err)
}

for k, v := range data {
		switch v.(type) {
		case map[string]interface{}:
			keys := []string{}
			for k := range v.(map[string]interface{}) {
				keys = append(keys, k)
			}
			fmt.Println(keys)
		}
		fmt.Printf("key : %T, value : %T\n", k, v)
	}
```

**※ 인터페이스 변수 사용 시 유의점**

```go
func main() {
  var inf_1 interface     // 이 때, inf_1 의 기본값은 nil 

  inf_1.Method()       // nil 상태에서 메서드를 실행하면 컴파일 에러 발생
  // inf_1.Method undefined (type infName has no field or method Method)

  Testing(inf_1)       // nil 상태에서 인자로 할당 시 런타임 에러 발생
  // panic: runtime error: invalid memory address or nil pointer dereference

**즉, interface 변수의 기본값 nil 이므로 아무것도 할 수 없음,
		사용하려면 인스턴스를 우선적으로 할당해야 함.** 
```
<br>

**Embedding Interface**

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}
type Writer interface {
  Write(p []byte) (n int, err error)
}

type ReadWriter interface {
	Reader
	Writer
}
/*
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
*/
```

- 임베딩 인터페이스 예시
    
    [ net.Error ]
    
    ```go
    type Error interface {
      error
      Timeout() bool
      Temporary() bool
    }
    ```
    
    [ heap.Interface ]
    
    ```go
    type Interface interface {
      sort.Interface
      Push(x interface{})
      Pop() interface{}
    }
    ```
    

**Tip.**  sort.Reverse()

사용자가 Interface 인터페이스로부터 생성한 구체화된 객체를 reverse로 객체를 재정의하여 사용

```go
type Interface interface {
	Len() int
	Less(i, j int) bool
	Swap(i, j int)
}
type reverse struct {
	// This embedded Interface permits Reverse to use the methods of
	// another Interface implementation.
	Interface
}

// Less returns the opposite of the embedded implementation's Less method.
func (r reverse) Less(i, j int) bool {
	return r.Interface.Less(j, i)
}

func Reverse(data Interface) Interface {
	return &reverse{data}
}

func (r reverse) Less(i, j int) bool {
	return r.Interface.Less(j, i)
}
```
