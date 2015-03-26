title: leetcode_08 String to Integer (atoi)
date: 2015-03-26 14:56:55
tags: leetcode
---

[leetcode problem site](https://leetcode.com/problems/string-to-integer-atoi/)

The requirements are as follows:

* 0s cannot be in front of a sign
* sign can appear at most once
* there shouldn't be spaces between a sign and the real values
* the function will try to interpret as much numbers as it can, which means if a character other than numbers appear after a continuous number sequence, the rest of the string will be ignored, and will return the value interpreted so far
* if overflow happens, return `INT_MAX if` it's positive, or `INT_MIN` if negtive
* if any law above is violated, 0 is returned

~~~ C++

class Solution {
public:
    int atoi(string str) {
        int len = str.length();
        int sign_count = 0;
        int index=0;
        int sign=1;
        // skip white spaces at the front
        while(str[index]==' ') index++;
        // skip sign
        while(str[index]=='+' || str[index]=='-') {
            if(str[index]=='+') sign_count++;
            else if(str[index]=='-') {
                sign_count++;
                sign *= -1;
            }
            // if multiple signs appears, imediately stop
            if(sign_count>1) return 0;
            // if there're spaces after sign
            if(sign_count>0 && str[index]==' ') return 0;
            index++;
        }
        // again, skip '0's
        while(str[index]=='0') index++;
        // the following is not numbers 
        // (if all string contents are already read, str[index]=='\0', which is also included here.)
        if(str[index]<'0' || str[index]>'9') return 0;
        // now, the rest should be numbers that construct a none-zero integer.
        // if anything other than 0~9 appears, return 0;
        // The rest characters that are not numbers are ignored.
        // Carefull about overflow.
        long long absolute=0;
        while(index<len && '0'<=str[index] && str[index]<='9'){
            absolute = absolute*10 + str[index++]-'0';
            // overflow, don't do this check at the end,
            // because long long absolute can also be overflowed...
            if(absolute>INT_MAX || absolute<INT_MIN) return sign>0?INT_MAX:INT_MIN;
        }
        
        // if not overflowed
        return (int)(sign*absolute);
    }
};

~~~
