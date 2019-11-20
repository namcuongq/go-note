# go-note
Vài điều cần ghi nhớ về golang (-.-)

## Contents

- [Code](#code)
- [Concurrency](#concurrency)
- [Performance](#performance)
- [Misc](#misc)
- [References](#references)

### Code
- [ ] sử dụng `go fmt` để làm đẹp code
- [ ] sử dụng `_` để bỏ qua thông số trả về
```go
  _ = f()
```
- [ ] duyệt các phần tử của mảng bằng cách sử dụng `range`
  -  thay thế `for i := 3; i < 9; i++ {...}` bằng `for _, c := range a[3:9] {...}`
- [ ] sử dụng \` cho chuỗi nhiều dòng
- [ ] rút ngắn các thông số cùng loại
```go
func f(a int, b int, s string, p string)
```
```go
func f(a, b int, s, p string)
```
- [ ] cẩn thận với struct giỗng, chi tiết: https://github.com/golang/go/issues/23440
```go
 func f1() {
    var a, b struct{}
    print(&a, "\n", &b, "\n") // Địa chỉ print ra giống nhau
    fmt.Println(&a == &b)     // Kết quả so sánh False
  }
```
- [ ] wrap errors với http://github.com/pkg/errors
```go
errors.Wrap(err, "tên chức năng lỗi")
```
- [ ] cẩn thận khi sử dụng `range` trong Go:
  - `for i := range a` and `for i, v := range &a` sẽ không copy mảng a
  - nhưng `for i, v := range a` sẽ copy mảng a
  - chi tiết: https://play.golang.org/p/Ta0q8Z0WJZS
- [ ] đọc key không tồn tại từ `map` không bị panic
  - value, found := map["no_key"] // found = false -> map không tồn tại key
- [ ] không sử dụng param thô
  - thay vì dùng số cụ thể like os.MkdirAll(root, 0700)
  - nên sử dụng hằng số giống như os.FileMode
- [ ] sử dụng iota
  - chi tiết: https://play.golang.org/p/NEpyw07ts9
- [ ] đế ngăn so sánh structs thêm 1 trường trống của loại `func`
```go
type Point struct {
	_ [0]func()
	X, Y float64
}
```
- [ ] di chuyển `defer` lên đầu
  - giúp đọc code nhanh hơn và dễ thấy những thứ được thực hiện ở cuối hàm

### Concurrency

- [ ] để block mãi mãi hãy sử dụng `select {}`

### Performance

- [ ] luôn luôn đóng http body `defer r.Body.Close()`
- [ ] không sử dụng `defer` trong vòng lặp
- [ ] đừng quên dừng ticker
```go
ticker := time.NewTicker(1 * time.Second)
defer ticker.Stop()
```
- [ ] sử dụng marshaler để tăng tốc marshaling
```go
type Entry map[string]string

func (e Entry) MarshalJSON() ([]byte, error) {
	buffer := bytes.NewBufferString("{")
	first := true
	for key, value := range e {
		jsonValue, err := json.Marshal(value)
		if err != nil {
			return nil, err
		}
		if !first {
			buffer.WriteString(",")
		}
		first = false
		buffer.WriteString(key + ":" + string(jsonValue))
	}
	buffer.WriteString("}")
	return buffer.Bytes(), nil
}

func main() {
	m := map[string]string{
		"key1":  "value1",
		"key2": "value2",
	}
	v, _ := Entry(m).MarshalJSON()
	fmt.Printf("%v\n", string(v))
}
```
- [ ] thay đổi regular expressions
  - để tăng hiệu suất trong các chương trình đồng thời nên tạo ra 1 bản sao
```go
re, err := regexp.Compile(pattern)
re2 := re.Copy()
```
- [ ] có 2 cách để `xóa map`
  - cách 1
  ```go
  for k := range m {
		delete(m, k)
	}
  ```go
  - cách 2
  ```
  m = make(map[int]int)
  ```
  
### Misc
  
  - [ ] đồng bộ struct
```go
var hits struct {
    sync.Mutex
    n int
  }
  hits.Lock()
  hits.n++
  hits.Unlock()
```

### References

- [ ] tổng hợp câu hỏi và câu trả lời về golang: https://github.com/goquiz/goquiz.github.io
