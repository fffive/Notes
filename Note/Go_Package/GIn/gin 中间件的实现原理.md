## gin 的中间件

| index        |              |              |           |
| ------------ | ------------ | ------------ | --------- |
| middleware 1 | middleware 2 | middleware 3 | operation |
> 当你正在执行方法时候，会将他放在所有中间件的后面。并且中间件会拥有 index 会移动指向正在执行的中间件。**所以当你 return 某一个 middleware 并不影响其他中间件**， 因为 gin 会将其移动。

