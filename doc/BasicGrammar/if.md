# if 判断

## 普通判断

```go
    if a > b {
        fmt.Println("a大于b")
    }
```

## 先赋值，再判断

```go
    if a:=9; a > b {
        fmt.Println("a大于b")
    }

    //or
    
    if a:=9; a > b {
        fmt.Println("a大于b")
    }else if a > c {
        fmt.Println("a大于c")
    }

```