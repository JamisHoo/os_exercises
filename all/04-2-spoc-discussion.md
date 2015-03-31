#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象

```
证明：
令S(t)是大小为n的栈中的内容，S’(t)是大小为n+k的栈中的内容，其中k>0,b(t)为新入栈的元素。
若证明LRU不会存在belady现象，则只需证对于任意t，S(t)属于S’(t)，且任意S(t)中元素a，对应到S‘(t)中元素a1，满足a的位置小于等于a1的栈位置。
使用数学归纳法：在初始情况时，S(t)和S‘(t)均为空，显然成立。假设i<=t-1时成立，则：
> 1 b(t) 属于 s(t),又属于 s‘(t)时，S(t)和S'(t)都不发生变化。成立。
> 2 b(t) 不属于 s(t),但属于 s‘(t)时，替换后肯定有b(t) 属于 s(t)，所以s(t)包含于s‘(t)，成立。
> 3 b(t) 不属于 s(t),也不属于 s’(t)时，s(t-1)包含于s’(t-1)，我们假设进行替换的元素是x，为最长的未访问时间的元素，也就是说x位于s‘栈的栈底。
如果x同时也被s’替换掉（x是s‘的栈底），本来包含于s‘（t-1）的s（t-1）两者同时减去一个元素。
加上b（t）就是新的，s（t）和s’（t）；如果x不在s’栈的栈底，那么s‘的栈底y拥有比x更久的未访问时间。
即y是要被替换的元素，同上，这个时候包含关系仍然成立。
因此LRU不会存在belady现象。
```


(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)
 根据课堂小组讨论结果，实现LRU算法代码如下：
 刘鹤 2012011300
 同组成员：李天润 2012011320
 ```
 #include <iostream>
 #include <limits>
 #include <list>
 int main(int, char**) {
    constexpr int working_set_window = 4;

    std::list< std::pair<int, int> > working_set; 
    int time = 1;
	
    working_set.push_back({1, 0});
    working_set.push_back({4, -1});
    working_set.push_back({5, -2});

    while (1) {
	// input every memory access from stdin
        int page_num;
        std::cin >> page_num;

        bool already_exist = 0;
		// if the page already exists in the memory, update the time
        for (auto ite = working_set.begin(); ite != working_set.end(); ++ite) {
            if(ite->first == page_num){
                already_exist = 1;
                ite->second = time;
            }
        }
	
	// if the page not exists in the memory
        if (!already_exist) {
            std::cout << "Page miss. " << std::endl;
            int last = std::numeric_limits<int>::max();
            int which = -1;
			// get the least recently used page
            for (auto ite = working_set.begin(); ite != working_set.end(); ++ite)
                if(ite->second < last){
                    last = ite->second;
                    which = ite->first;
                }
			// swap out this page
            for(auto ite = working_set.begin(); ite != working_set.end(); ++ite)
                if(ite->first == which){
                    ite->first = page_num;
                    ite->second = time;
                }
        } else {
            std::cout << "Page hit. " << std::endl;
        }
	// show the memory after every memory access
        for (const auto i: working_set)
            std::cout << "(" << i.first << ", " << i.second << ") ";
        std::cout << std::endl;
        ++time;
    }
}

 ```
 
## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
