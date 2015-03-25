title: leetcode_02 Add two numbers
date: 2015-03-25 18:15:45
tags: leetcode
---


[leetcode problem site](https://leetcode.com/problems/add-two-numbers/)

I'm working on projects in functional programming languages these days, so recursion is used in this case.

Need to be causious when dealing with input cases like {0, 0} and numbers with a length difference larger than 1. 

Then specify the stop case of the recursion where the two input nodes are both NULLs, but don't forget there still may be a non-zero complement value. 

~~~ C++

/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
        return addTwoNumbers(l1, l2, 0);
    }
    
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2, int complement) {
        if(l1!=nullptr && l2!=nullptr){
            int sum = l1->val + l2->val + complement;
            ListNode *pNode = new ListNode(sum%10);
            pNode->next = addTwoNumbers(l1->next, l2->next, sum/10);
            return pNode;
        }else if(l1==nullptr && l2!=nullptr){
            int sum = l2->val + complement;
            ListNode *pNode = new ListNode(sum%10);
            pNode->next = addTwoNumbers(nullptr, l2->next, sum/10);
            return pNode;
        }else if(l1!=nullptr && l2==nullptr){
            int sum = l1->val + complement;
            ListNode *pNode = new ListNode(sum%10);
            pNode->next = addTwoNumbers(l1->next, nullptr, sum/10);
            return pNode;
        }else{ // There's no number left to add, then add a complement node
               // if needed.
            return complement?(new ListNode(1)):nullptr;
        }
    }
};

~~~