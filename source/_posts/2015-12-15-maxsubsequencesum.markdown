---
layout: post
title: "最大子序和的联机解法"
date: 2015-12-15 17:42
comments: true
categories: 
---
##最大子序和的问题##

####问题描述：  
给定整数A<sub>1</sub>,  A<sub>2</sub> ,...  A<sub>N</sub>, 求 $$\sum_{k=i}^{j}A_k$$的最大值（**为方便起见，如果所有整数为负数，则最大子序和为0**）

下面直接给出最后的算法，直接的解法可以看其它人的博客

    int MaxSubSequenceSum(const int A[], int N )
    {
        int CurtSum = 0;
        int MaxSum  = 0;  
        
        for (int j =0; j<N;j++)
        {
            CurtSum += A[j];
            if(CurtSum > MaxSum)   
                MaxSum = CurSum;
            else if( CurtSum<0 )
                CurSum = 0 ;
    	}  

        return MaxSum;
    }
    

那这个算法为什么正确呢？     
假设有如下序列：  

>3, -2 , 3 , -100 , 4 , -2  

如果按照上面的算法，其实是计算了两个子序列  
A[0]~A[2] 累加为 4， 遇到A[3]=-100，序列和小于0，CurSum从0开始重新计算，A[4]~A[5]和为2,  
然后取这两个子序列大的为MaxSum。  
那以A[1]为开头的子序列你一个都没算过，你怎么知道最大的不在其中呢？   

有个关键点是：  
> 定理1：如果一个序列的和为负数，那么这个序列肯定不是另外一个**最大子**序列的前缀

我们假设如下：   

$$\sum_{k=i}^{j-1}A_k >=0$$  

$$\sum_{k=i}^{j}A_k <0 $$  
   
也就是说A[j] 是使$$\sum_{k=i}^j$$为负的那个元素。  
根据定理1有如下推论：
**如果存在一个最大子序列是以A[i]开头的，那么最后一个元素的下标肯定是小于j的**
  
  
假设**i<p<j**

那么我们有如下结论：  

$$\sum_{k=p}^qA_k$$ （q<n）   **<=**   $$\sum_{k=i}^qA_k$$ （i<p,q<n）  

**因为i和p之间的元素和肯定是大于0的**      
也就是说从p开始的任意子序和都不大于以A[i]开始并包含它的子序列的和。   
那根据前面的推论，如果有以A[i]开始的最大子序列，最后一个元素的下标识小于j的。


<div id="disqus_thread"></div>
<script>
/**
* RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
* LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL; // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');

s.src = '//merlinli.disqus.com/embed.js';

s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
