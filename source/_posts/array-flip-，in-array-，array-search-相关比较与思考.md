---
title: array_flip()，in_array() ，array_search()相关比较与思考
date: 2021-06-30 23:24:56
tags: [PHP, 效率, 函数]
categories: [PHP]
---

### `array_flip()`，`in_array()` ，`array_search()`相关思考

在**1.两数之和**题目中自己遇到这三个函数的相关疑问，这篇做一个相应的解答。

首先，由文章 https://www.php.cn/php-weizijiaocheng-432375.html

可得相关效率问题，`array_flip()` 效率比 `in_array()` 和 `array_search()` 高（一般情况下，其实我看LearnKu中有人也分析说是后面效率比前面的高，具体的就先以这个为准吧！）。

但是在上一题中，用到`array_flip()`发现效率居然比`array_search()`使用时间更长，后面发现问题所在，其实是在循环中，不断的`array_flip()`导致了一定的耗时问题。

```php
function twoSum(array $nums, int $target): array
{
    for ($i = 0; $i < count($nums);$i++) {
        // 计算剩下的数
        $residue = $target - $nums[$i];
        // 匹配的index，有则返回index， 无则返回false
        $reverse_nums = array_flip($nums);
        $is_exist = array_key_exists($residue, $reverse_nums);
        if ($is_exist !== false && $reverse_nums[$residue] != $i) {
            return array($i, $reverse_nums[$residue]);
        }
    }
    return [];
}
```

再者就是接下来的思考，因为观看其他耗时比较少的代码，发现他们其实用到的foreach循环达到反转，再进行后面的操作，提高了效率，这时便抛出相关疑问：难道foreach比`array_flip()`还要快？

于是，自己进行了相应的对比，分别用短数组和长数组进行相关测试。

代码如下：

~~~php
```
<?php
// 短数组比较
$short_array = [10, 22, 13, 4, 45];
$start_time = microtime(true);
foreach ($short_array as $key=>$value) {
    $new_array1[$value] = $key;
}
$end_time = microtime(true);
echo '短数组比较-for循环所消耗时间：'. ($end_time - $start_time)."\n";

$start_time = microtime(true);
$new_array2 = array_flip($short_array);
$end_time = microtime(true);
echo '短数组比较-array_flip所消耗时间：'. ($end_time - $start_time)."\n";


// 长数组比较
$long_array = [];
for ($i = 1; $i < 5000; $i++) {
    array_push($long_array, $i);
}
$start_time = microtime(true);
foreach ($long_array as $key=>$value) {
    $new_array3[$value] = $key;
}
$end_time = microtime(true);
echo '长数组比较-for循环所消耗时间：'. ($end_time - $start_time)."\n";

$start_time = microtime(true);
$new_array4 = array_flip($long_array);
$end_time = microtime(true);
echo '长数组比较-array_flip所消耗时间：'. ($end_time - $start_time)."\n";
```
~~~

时间对比：

![](https://gitee.com/Gardenian/blog-image/raw/master/img/20210630/R$K$8FW3O5%FQWFT$~~F9$0.png)

不难发现，短数组情况下对于`array_flip()`比循环快不了多少，而遇到较大数组的时候，循环明显比array_flip快许多。这里在后面编写其他代码中需要注意。