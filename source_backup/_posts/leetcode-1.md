title: leetcode_01 Two sum
date: 2015-03-25 17:18:58
tags: leetcode
---

[leetcode problem site](https://leetcode.com/problems/two-sum/)

The easiest and slowest solution is very obvious, which has O(n^2) time complexity. Pseudo-code is as follows:

~~~ 

foreach elementA in numbers:
    foreach elementB in all elements on the left side of elementA:
        if elementA+elementB == target:
            return a vector with indexA+1, indexB+1

~~~

This solution will be judged time out by Leetcode oj.

My first attempt is as follows, which has O(nlogn) complexity. Pseudo-code is as follows:

~~~ 

sortedNumbers = sort(numbers)
indexL, indexR = 0, len(numbers)-1

while(indexL!=indexR):
    L, R = sortedNumbers[indexL], sortedNumbers[indexR]
    s = L + R
    if s >target:
        R--
    elif s < target:
        L++
    else:
        return a vector with (L+1, R+1)

~~~

Then inspired by my friend, using Hash table will reduce the complexity to O(n). The cxx code is as follows:

~~~ C++

class Solution {
public:
    vector<int> twoSum(vector<int> &numbers, int target) {
        unordered_map<int, int> hmap;
        vector<int> result;
        for(int i=0; i<numbers.size(); i++){
            int a = numbers[i];
            int toFind = target-a;
            if(hmap.find(toFind)!=hmap.end()) {
                result.push_back(hmap[toFind]+1);
                result.push_back(i+1);
                return result;
            }
            hmap[numbers[i]] = i;
        }
        return result;
    }
};

~~~
