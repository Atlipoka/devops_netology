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
