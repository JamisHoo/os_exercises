# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
```
- 最优匹配：  优点：大部分分配为较小尺寸时有很好的性能。缺点：释放较慢，会产生很多小碎片。
- 最差匹配：  优点：中等大小分配时有较好效果。缺点：会破坏较大的空闲分区。
- 最先匹配：  优点：实现简单，在高地址空间有较大空闲内存。缺点：进行大块分配时速度较慢。
- buddy system：  优点：避免将较大内存块无限拆碎。缺点：拆分以及合并操作会产生较大开销。 

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

```
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
-----详见代码
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
-----详见代码
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
-----对于要求分配和释放的内存大小，先对其判断是否四对齐。
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```

```
#include <iostream>
#include <vector>
#include <cstdlib>

using namespace std;

struct page{
	int start;
	int length;
	page(int s, int l) start(s), length(l){}
}

struct pmm_manager{
	vector<page> page_list;
	vector<page> noavailable_list;
	
	int max = 1000;
	
	pmm_manager(){
		srand(0);
	}
	
	void init(){
		int start;
		int length;
		for (int i=0; i<20; i++){
			length = 100*(i+1);
			page_list.push_back(page(start, length));
			start += length;
		}
	}
	
	void malloc(int length){
		int l = page_list.size();
		int i;
		for (i=0; i<l; i++){
			if (page_list[i] >= length)
				break;
		}
		int newLength = page_list[i].length-length;
		int newStart = page_list[i].start+length;
		page_list.erase(page_list.begin()+i);
		
		for (i=0; i<page_list.size(); i++){
			if (page_list[i] >= newLength)
				break;
		}
		page_list.insert(page_list.begin()+i-1, page(newStart, newLength));
		
		for (i=0; i<noavailable_list.size(); i++){
			if (noavailable_list[i].start >= newStart-length)
				break;
		}
		
		if (i == 0){
			noavailable_list.insert(noavailable_list.begin(), page(newStart-length, length));
		}
		else {
			noavailable_list.insert(noavailable_list.begin()+i-1, page(newStart-length, length));
		}
		
	}
	
	void free(int start){
		int i;
        for(i=0; i<noavailable_list.size(); i++)
            if(start == noavailable_list[i].start)
                break;
        if(i == noavailable_list.size()){
            return;
        }

        int sz = noavailable_list[i].length;
        noavailable_list.erase(noavailable_list.begin()+i);
        for(i = 0; i < page_list.size(); i++)
            if((page_list[i].start + page_list[i].length) == start || (start + sz) == page_list[i].start)
                break;
        page temp;
        if(i != page_list.size()-1){
            if(page_list[i].start > start){
                page_list[i].start = start;
                page_list[i].length += sz;
            }
            else
                page_list[i].length += sz;
            temp.length = page_list[i].length;
            temp.start = page_list[i].start;
            page_list.erase(page_list.begin()+i);
        }
        else
            temp.length = sz, temp.start = start;

        for(i = 0; i < pagelist.size(); i++)
            if(page_list[i].length > sz)
                break;
        page_list.insert(page_list.begin()+i, temp);	
	}
}

int main(){
	pmm_manager pm;
    pm.init();
    
    pm.malloc(800);
    
    pm.free(2333);
    
    return 0;
}
```

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



