# 何时使用panic

## 一、总体原则

在rust里错误重要分类两类，一类是可恢复的，另一类是不可恢复的，通过调用`panic!`宏代表该错误不可恢复。通过返回`Result`则相当于错误进行了传播，而这类错误是可恢复的。

当我们自己写的代码，在某些情况下决定为不可恢复错误的时候，我们可以使用`panic!`。如果我们写的函数，如果某些情况决定为错误可以恢复，可以返回`Resul`，这时候将错误的处理权交给了代码的调用者，代码的调用者要根据实际情况决定是否恢复此错误。总结来说如下：

- 在定义一个可能失败的函数时，优先考虑返回`Result`
- 如果某种情况出现的失败失败肯定不可恢复的，就使用`panic!`

## 二、使用使用`panic!`的场景如下

- 演示某些概念：使用unwrap函数，发生错误将使用`panic!`标记；
- 原型代码：unwrap、expect函数，在原型代码里还不知道如何错误错误，使用unwrap、expect这类方法将使用`panic!`标记错误，发生错误时便于跟踪问题；
- 测试：使用unwrap、expect函数，测试的代码如果出现错误，会被`panic!`立即停止。

## 三、错误处理的指导性建议

当代码最终可能处理损坏状态时，最好使用`panic!`。

损坏状态（bad state）表示某些假设、保证、约定或不可变性被打破，如下列情况都可以被认定为损坏状态：

- 非法的值、矛盾的值、或者空缺的值被传入了代码
- 这种状态并不能预测，而是偶尔发生
- 如果你的代码处理这种状态就无法运行
- 在你使用的数据类型中没有一个更好的方法来将这些（处于损坏状态）的信息进行编码

如果不理解以上的概述，我们可以参考一下几个建议使用`panic!`或者返回`Resul`的场景

- 当你的代码被调用时，传入了无意义的参数值：`panic!`
- 调用外部不可控代码，返回非法状态，你无法修复：`panic!`
- 如果失败是可以预期的：`Result`，比如我们想把一个字符串解析成数字，这时候可能成功，可能失败，是可预期的
- 当你的代码对某些之进行操作，首先验证这些值的合法性，如果不合法，进行`panic!`。如果输入数据不合法，往往是因为调用者逻辑出现了问题，也不应该由调用者进行解决，应该就地进行`panic!`

## 四、数据验证逻辑的优化

针对调用者可能传入非法值的情况，我们可以进行逻辑优化，在创建我们需要的数据类型时，我们在构造函数里实现验证逻辑，我们则无需担心接收的值的有效性。如在该系列文章的[“猜数游戏”](../02_guessing_game/01_guessing_game.md)中，可以有以下逻辑，验证逻辑

```rust
// ... 前面的逻辑忽略掉
// 先将guess变量转换为整数类型
let guess: i32 = match guess.trim().parse() {
    // 解析正确时将数字返回
    Ok(num) => num,
    // 解析错误时，使用 continue 跳出本次循环，进行下一次循环
    Err(_) => continue,
};

// 验证传入数值必须大于1、小于100
if guess < 1 || guess > 100 {
    println!("The secret number will be between 1 and 100");
    continue;
}

// ... 后面的逻辑忽略掉
```

我们改为以下逻辑

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    // 定义一个关联函数，相当于构造函数
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("The secret number will be between 1 and 100");
        }

        Guess { value }
    }

    // 定义获取值的方法
    pub fn value(&self) -> i32 {
        self.value
    }
}

fn main() {
    loop {
        // ...省略前面的逻辑

        // 先将guess变量转换为整数类型
        let guess: u32 = match guess.trim().parse() {
            // 解析正确时将数字返回
            Ok(num) => num,
            // 解析错误时，使用 continue 跳出本次循环，进行下一次循环
            Err(_) => continue,
        };

        let guess = Guess::new(guess);

        // ...省略后面的逻辑
    }
}
```

在以上代码中，我们将解析的数据传如`Guess::new`，如果不发生`panic!`，代码数据合法，通过了验证；如果发生了`panic!`，说明传入数据没有通过验证。使用此方式进行数据，优化了验证的过程，无需把验证的逻辑到处写，提高了代码的可读性。

要注意的是：在上面的`Guess`结构体中，必须把`value`属性设置为私有，再通过结构体方法获取属性的数据。我们不允许通过`Guess`结构体直接修改`value`的值，在模块外的代码必须使用`Guess::new`函数来创建`Guess`的实例。这样能确保`value`值通过了有效性的检查，通过检查则产生实例，否则则发生`panic!`。
