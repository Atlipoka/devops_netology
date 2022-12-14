Перевод метров в футы
```
package main

import "fmt"

func main() {
	var input float64
	input = 5
	fmt.Print("Quantity in metros: ", input)
	output := input / 0.3048
	fmt.Print(" and we convert this in feets: ", output)
}
```

Минимальный елемент в массиве
```
package main

func main() {
	x := []int{48, 96, 86, 68, 57, 82, 63, 70, 37, 34, 83, 27, 19, 97, 9, 17}
	y := x[0]
	for _, min := range x {
		if min < y {
			y = min
		}
	}
	println(y)
}
```
Числа от 1 до 100 кратные 3
```
package main

import "fmt"

func main() {
	s := []int{}
	for i := 0; i <= 100; i++ {
		if i%3 == 0 {
			s = append(s, i)
		}
	}
	fmt.Println(s)
}
```
