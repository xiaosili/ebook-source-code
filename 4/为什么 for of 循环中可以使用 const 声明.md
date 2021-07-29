#　为什么 for of 循环中可以使用 const 声明

有人认为，for of 循环中迭代元素的变量是同一个，在迭代前声明，所以要使用 let 声明

实际上，每次迭代都会声明一个新的变量，它继承了元素的值，_这与 for (;;) 循环相似_

那么，怎样证明 for of 循环每次迭代内都会声明一个新的变量？

[mdn Symbol.iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator)
[mdn Array.prototype.values()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/values)
[mdn Array.prototype[@@iterator]()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/@@iterator)
## for of 循环的等价写法

首先，我们要知道 for of 循环的等价写法

```javascript
const arr = [1, 2, 3];
for (const item of arr) {
  console.log(item);
}
// for of 循环的等价写法
{
  const iterator = arr[Symbol.iterator]();
  let item;
  while (!(item = iterator.next()).done) {
    console.log(item.value);
  }
}
```

## 宏任务访问同一变量

其次，我们通过宏任务延迟访问同一变量，它的值是相同的

```javascript
// 宏任务访问同一变量的值相同
{
  let i = 0;
  while (i < 3) {
    i++;
    setTimeout(() => {
      console.log(i); // 3 3 3
    }, 0);
  }
}
```

## for of 循环内宏任务的等价写法

然后，我们在 for of 循环内调用宏任务访问迭代元素，发现输出结果并不相同

```javascript
const arr = [1, 2, 3];
for (const item of arr) {
  setTimeout(() => {
    console.log(item); // 1 2 3
  }, 0);
}
```

假设，for of 循环内的迭代元素为同一变量，那么上面的等价写法应该是怎样的呢？

```javascript
const arr = [1, 2, 3];
{
  const iterator = arr[Symbol.iterator]();
  let item;
  let value;
  while (!(item = iterator.next()).done) {
    value = item.value;
    setTimeout(() => {
      console.log(value); // 3 3 3
    }, 0);
  }
}
```

然而，实际上 for of 循环内执行宏任务的等价写法是：

```javascript
const arr = [1, 2, 3];
{
  const iterator = arr[Symbol.iterator]();
  let item;
  while (!(item = iterator.next()).done) {
    const value = item.value; // 这里可以使用 const
    setTimeout(() => {
      console.log(value); // 1 2 3
    }, 0);
  }
}
```
