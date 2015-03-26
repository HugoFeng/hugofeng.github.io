title: leetcode_07 Reverse Integer
date: 2015-03-26 12:00:15
tags: leetcode
---

[leetcode problem site](https://leetcode.com/problems/reverse-integer/)

This is an easy one. Special cases like 0 should be considered, and most importantly, OVERFLOW should also be taken into account, which I always miss.

~~~ C++

class Solution {
public:
    int reverse(int x) {
        if(x==0) return 0;
        // store the sign, make x non-negtive
        int sign = 1;
        if(x<0){
            sign = -1;
            x *= -1;
        }
        // avoid overflow
        long long result = 0;
        while(x){
            result = result*10 + x%10;
            x /= 10;
        }
        // if there's a overflow
        if(result!=(int)result) return 0;
        return result*sign;
    }
};

~~~