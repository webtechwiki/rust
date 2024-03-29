# 模式匹配可失败性

模式的两种形式包括：可辩驳的（可反驳的或可失败的）、无可辩驳的（不可反驳的或不可失败的）。

能够匹配任何可能传递的值的模式是无可辩驳的，即不能失败，怎么匹配都会成功。例如`let x = 5`不可能失败，它能匹配所有表达式右侧的值。

对于某些可能的值，无法进行匹配的模式，就是可辩驳的。例如`if let Some(x) = a_value`，如果右边的值是null，这时候就会发生不匹配的情况，这就是可辩驳的，或者可失败的。

其中，函数参数、let语句、for循环只接受无可辩驳的模式。`if let`和`while let`接受可辩驳和无可辩驳的模式。在接受无可辩驳模式的时候编译器可能会发生警告，因为存在可能的失败，如下例子

```rust
fn main() {
    let a: Option<i32> = Some(5);
    // Some是个可辩驳的
    let Some(x) = a;
}
```

a是Option类型，而Some是一个可辩驳的，以上代码将会编译错误，可以修改为如下代码

```rust
fn main() {
    let a: Option<i32> = Some(5);
    if let Some(x) = a {

    }
}
```

但是如果我们改为不可辩驳的模式，如下

```rust
fn main() {
    let a: Option<i32> = Some(5);
    if let x = 5 {}
}
```

代码不会报错，但是`x = 5`总是匹配成功的，所以使用一个可辩驳的模式没有任何意义。结合上面的示例，我们可以想到`match`表达除了最后一个分支的所有分支必须是可辩驳的（即可失败的），而最后一个分支因为是不可辩驳的，因为它需要匹配所有剩余的情况。
