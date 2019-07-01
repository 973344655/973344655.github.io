---
title: 滑动窗口解决最长子串问题
date: 2019-5-29 14:25:33
tags: [algorithm]
---

#### 1.问题
寻找一个字符串中最长不重复的字符串长度。<br>
(1)通用做法:<br>
```
for(int i=0; i<len; i++){
    for(int j=i; j<len; j++){
      //找到以i开头的最长不重复子串
      if(){
        break;
      }
    }
}
```
(2)用滑动窗口来解决<br>

#### 2.滑动窗口
原理：<br>
维护一个长度为right-left的窗口，通过左右边界的收缩与拓展来找到最大长度。<br>
只需一次遍历，时间复杂度O(n).<br>

#### 3.实现
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        //滑动窗口
        int left = 0, right = 0;
        int max = 0;
        Set window = new HashSet();
        while(left<s.length() && right<s.length()){
            if(!window.contains(s.charAt(right))){
                window.add(s.charAt(right));
                right++;
                //更新最大值
                max = Math.max(max,right-left);
            }else{
              //收缩左边界到没有重复为止，
              //按照顺序收缩，例 abcb->bcb->cb
                window.remove(s.charAt(left));
                left++;
            }

        }
        return max;
    }
}
```
