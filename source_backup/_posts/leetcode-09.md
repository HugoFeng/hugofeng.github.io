title: leetcode_09 Palindrome Number 
date: 2015-03-26 20:16:55
tags: leetcode
---

[leetcode problem site](https://leetcode.com/problems/palindrome-number/)

It is required not to use additional memory, so converting the integer to a string is not possible. Here I'm using recursion.

There're two lines need to be dealt carefully:

* Line 16: Cases such as `1000021` need the length of the sub-sequence of digits be mesured by variable `len` in stead of the value of the sub-sequence.
* Line 20: the recursive function call should be placed after the first predicate, so that if the first is false, the function won't be called.

~~~ C++

class Solution {
public:
    bool isPalindrome(int x) {
        // case when x is 0 or negtive
        if(x<0) return false;
        // case when x has one digit
        else if(x<10) return true;
        // number of digits in x
        int len = intLen(x);
        return isPalindrome(x, len) ;
    }
    
    bool isPalindrome(int x, int len) {
        // Since this is a sub-sequence of digits of the original number,
        // zeros might be leading this sequence, so use "len" instead of x<10 to judge.
        if(len<2) return true;
        int low = x%10;
        int devider = pow(10, len-1);
        int high = x/devider;
        if(high==low && isPalindrome((x%devider-low)/10, len-2)) 
            return true;
        else 
            return false;
    }
    
    int pow(int base, int n){
        int result = 1;
        if(n==0) return 1;
        while(n--) result*=base;
        return result;
    }
    
    int intLen(int input){
        int len = 0;
        while(input){
            input /= 10;
            len++;
        }
        return len;
    }
};


~~~