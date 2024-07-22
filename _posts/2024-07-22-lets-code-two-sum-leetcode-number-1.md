---
layout: post
title: Let's Code Two Sum - LeetCode Number 1
excerpt_separator: <!--more-->
---

Recently I worked through [Two Sum](https://leetcode.com/problems/two-sum/description/), a classic DSA problem and the first problem on LeetCode's website. I used PHP to code the solution since that's the language I use at my day job.<!--more-->

Here's the prompt:

> Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`. You may assume that each input would have **<em>exactly one solution</em>**, and you may not use the same element twice. You can return the answer in any order.


### Initial Thoughts

My first thought was, *what are the different ways to get a single value from an array?* I can check the array values individually, but that would be a brute-force approach. If my goal is efficiency, then there needs to be another way. Looping through the array inside of nested for loops would have a time complexity of n squared, which is not efficient. My second thought was that I could sort the array, and this would place numbers near each other. But without something more, this would still require looping through the array with nested for loops, which still results in a time complexity of N squared.

Ultimately, I did not figure out the solution on my own. I got help from [Neetcode](https://www.youtube.com/watch?v=KLlXCFG5TnA), where he explains that Two Sum is just a basic algebra problem.

Since I always have the 'target' value and at least one other value from the array, I have a simple formula: `x = (target - y)`, where y is the value from the current iteration of the array. That means I don't need to search through the entire array. I simply need to search for `x`.

So how do I find `x` without looping through the entire array? I already know that hashing results have a lookup time of O(1), which is constant time. So if I put all values in the array inside of a hash map or a hash set, then I can access `x` without looping through each entry in the array. 

If I store the index of each number as the value, then I have the result once x is found. And since I'm only allowed to use each element once, that means each number from the input array must be unique in the final result. Effectively, I need all of the array keys to be values, and the values to be keys. PHP has a built-in `array_flip` function to achieve this. But for the sake of working through the problem, and to follow the Neetcode example I previously linked to, I'm not going to use `array_flip`. 

### Approach

Now to implement this. *How do I turn all of the array's values into keys, and its keys into values?* One approach is to loop through the original array, adding the keys one by one.

Something like this:

```
For each entry in the nums array
    key equals the value from nums
    value equals the key from nums
    Add the key/value pair to a hash map
```

Lastly, I need to check if `x` exists as a key inside the new hashmap. If it does, I found our result and I can exit the function. 

Here's the implementation:

```php
function twoSum($nums, $target) {
    $result = [-1, -1];
    $hashMap = [];

    for($i = 0; $i < count($nums); $i++) {
        $key = $nums[$i];
        $hashMap[$key] = $i;

        $searchValue = $target - $key;

        if(array_key_exists($searchValue, $hashMap)) {
            $result = [$i, $hashMap[$searchValue]];
            break;
        }
    }

    return $result;
}
```

My code resulted in an error on the following test case: `[3, 2, 4]` where `target = 6`. Since the last two numbers equal 6, this means there is an edge case that I didn't think of.

I failed to consider the possibility of `searchValue` equaling the current value of the array. If this happens, then the code will exit early. To fix this, I can alter the approach by storing multiple keys. 

Because `searchValue` and the current value are keys, then I added an aditional check to make sure they're unique. This should prevent the code from ending early in the edge case where the `searchValue` is the same as the current value: In other words, I checked whether or not the current index was previously added into the hashMap:

```php
    function twoSum($nums, $target) {
        $result = [-1, -1];
        $hashMap = [];

        for($i = 0; $i < count($nums); $i++) {
            $key = $nums[$i];
            $hashMap[$key] = $i;
            $searchValue = $target - $key;

            if(array_key_exists($searchValue, $hashMap)) {
                $notUnique = ($i == $hashMap[$searchValue]);

                if($notUnique) {
                    continue;
                } else {
                    $result = [$i, $hashMap[$searchValue]];
                    break;
                }
            }
        }

        return $result;
    } 
```
    
However, my new solution does not pass when the input array gives two values that equal each other, such as `[3, 3]` where `target = 6`. I puzzled over this for some time. My first thought was that I introduced a bug into my code. But in reality, my solution was incorrect from the beginning. 

There's no need to have a conditional check to see if the search value was previously added as a key into the hashmap. All I needed to do was reorder the code. On each iteration of the loop, I should get the search value. Then, I should check to see if it already exists in the hashMap. If it does, then I can exit the function with the result. If not, I need to add it to the hash map and continue the loop. 

### Solution

```php
class Solution {

    /**
     * @param Integer[] $nums
     * @param Integer $target
     * @return Integer[]
     */
    function twoSum($nums, $target) {
        $result = [-1, -1];
        $hashMap = [];

        for($key = 0; $key < count($nums); $key++) {
            $value = $nums[$key];
            $searchValue = $target - $value;

            if(array_key_exists($searchValue, $hashMap)) {
                $result = [$key, $hashMap[$searchValue]];
                break;
            }

            $hashMap[$value] = $key;
        }

        return $result;
    }
}
```