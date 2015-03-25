title: leetcode_03 Longest Substring Without Repeating Characters 
date: 2015-03-25 21:10:55
tags: leetcode
---

[leetcode problem site](https://leetcode.com/problems/longest-substring-without-repeating-characters/)


~~~ C++

class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int length = s.length();
        char *array = new char[length+1];
        strcpy(array, s.c_str());
        
        int currentLen = 0;
        int maxlen = currentLen;
        int tail = 0;
        
        unordered_map<int,int> hash;
        for(int i=0;i<length; i++){
            char c=array[i];
            // if this character appeared before
            if(hash.find(c)!=hash.end()){
                int usedTobe = hash[c];
                hash[c] = i; // update the latest occurence of this character
                // if this character has seen within the latest 
                // non-duplicated substring.
                if(usedTobe>=tail){
                    // Since a duplicate is seen here, 
                    // conclude the last substring.
                    // If current substring is the longest ever seen, update
                    if(currentLen>maxlen) 
                        maxlen=currentLen;            
                    // Exclude the previous occurence
                    tail = usedTobe+1; 
                }
            // if this is a new character ever seen
            }else {
                hash[c] = i;
            }
            
            // update the length of current substring in each iteration
            currentLen = i-tail+1;
        }
        return max(currentLen, maxlen);
    }
};


~~~


[A more elegant implementation](https://leetcode.com/discuss/13336/shortest-o-n-dp-solution-with-explanations)  by others.