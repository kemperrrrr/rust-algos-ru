# 1. Сумма двух
Ссылка на задачу: [Two Sum](https://leetcode.com/problems/two-sum/)

Каждый, кто решал LeetCode, навсегда запоминает эту задачу (как когда-то первое слово в учебнике английского — «abandon», ха-ха). Но написать её на Rust не так просто, особенно для новичков. Уверен, после разбора этой задачи вы глубже поймёте некоторые детали Rust.

## Решение 1
Простое и грубое: двойной цикл, перебор всех пар до нахождения подходящей.

```rust,no_run
# struct Solution;
impl Solution {
    pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
        for i in 0..nums.len() {
            for j in (i + 1)..nums.len() {
                if nums[i] + nums[j] == target {
                    // В Rust тип индекса — usize, поэтому приводим i, j к i32
                    return vec![i as i32, j as i32];
                }
            }
        }
        vec![]
    }
}
```

## Решение 2
Посмотрим на второй цикл for из первого решения:

```rust,ignore
for j in (i + 1)..nums.len() {
    if nums[i] + nums[j] == target {
        ...
    }
}
```

Попробуем иначе:

```rust,ignore
for j in (i + 1)..nums.len() {
    let other = target - nums[i];
    if nums[j] == other {
        ...
    }
}
```

Таким образом, наша цель — найти значение в массиве. Это отличная задача для HashSet/HashMap! Поскольку нам ещё нужно получить индекс соответствующего значения, используем HashMap.

```rust,no_run
# struct Solution;
use std::collections::HashMap;

impl Solution {
    pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
        let mut map = HashMap::new();
        for (index, value) in nums.iter().enumerate() {
            let other = target - value;
            if let Some(&other_index) = map.get(&other) {
                return vec![other_index as i32, index as i32];
            }
            map.insert(value, index);
        }
        vec![]
    }
}
```

Если вы новичок в Rust, у вас могут возникнуть следующие вопросы:

- Зачем нужна строка `use std::collections::HashMap;`? HashMap используется не так часто, поэтому Rust не включил её в Prelude. При необходимости её нужно подключать вручную.
- Куда делся `for i in 0..nums.len()` и что такое `for (index, value) in nums.iter().enumerate()`? В Rust рекомендуется использовать итераторы. Адаптер `enumerate` возвращает кортежи (индекс, значение).
- Почему в `if let Some(&other_index) = map.get(&other)` нужны два знака ссылки? Из-за системы владения. [Метод get](https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.get) принимает ссылку на ключ и возвращает `Option<&value>`, поэтому требуются две ссылки.
