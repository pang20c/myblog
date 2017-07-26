title: longest substring without repeat
date: 2017-07-21 19:55:57
tags:
- 算法
- 字符串
- leetcode
category:
- 算法
---

> Given a string, find the length of the longest substring without repeating characters.

> Examples:

> Given "abcabcbb", the answer is "abc", which the length is 3.

> Given "bbbbb", the answer is "b", with the length of 1.

> Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

## 获取一个字符串中无重复字符的最长字符串

```
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashMap<Character,Integer> map = new HashMap<>();
        int max = 0;
        int j = 0;
        for(int i = 0;i<s.length();i++){
            Character c = s.charAt(i);
            if(map.containsKey(c)){
                j = Math.max(map.get(c)+1,j);
            }
            map.put(c,i);
            max = Math.max(max,i-j+1);
        }
        return max;
    }
}
``` 
# 解析：
'fabcdaf' 字符串从0开始编号
    可以想象手握一根线的两头在字符串间游走，不断去丈量寻找最大的长度，右手用`i`代表，左手用`j`去表示，`Hashmap`是我们的记账本，每个字母的位置我们都记录在本子上，同时`i`每走一步我们就算一次最大长度算法就是左右手间的距离也就是`i-j+1`（其实没有必要每次都算但是这样的代码最简洁），对于重复出现的字母我们只记录最新的。
    让`i`先往右走 即 `i++`，并把`i`碰到的字母和位置记录到本子中，重复这个过程，直到本子上第一次出现了重复的字母，我们看一下本子上这个字母的位置，比如`i`现在走到了`5`碰到了重复字母`a`，本子上记录的位置是`1`，这时候我们就要挪动左手了，因为两个手之间出现了重复的字符我们要把它跳过去，那把左手往哪挪呢？肯定是往右挪，挪到哪最合适呢？当然是紧挨着1号位的右手边二号位是最合适的，因此这时`j`的取值就变成了`map.get(c)+1`，好，到了最难理解的一步`Math.max`的作用，这时候它的作用还没发挥出来，我们接着让`i`往右走。
    此时的`j`停在了`2`号位,`i`走到了字符`f`,`i`又一次的从本子上发现了出现过的字母，又该动动你的左手了，往哪边挪？不能往左，因为往左会出现更多的重复（a的重复）只能往右或者原地不动，而因为`f`的位置比`a`还要靠左，那好看来原地不动是最好的选择，所以我们在`j`的取值上增加了一个max处理，就是防止左手又回到了一个错误的位置上。
    
# 总结：
> i 和 j 这两个指针是只能往右走的 不能走回头路，i负责探路，j负责跳过路上的坑。 

 
### leetcode 链接
* https://discuss.leetcode.com/topic/8232/11-line-simple-java-solution-o-n-with-explanation
    



