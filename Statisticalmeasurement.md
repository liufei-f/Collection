[TOC]



# pFDR

The pFDR is the posterior probability of belonging to the null rather than the non-null component.

Storey and colleagues introduce a modified approach to FDR that differs from the original BH procedure in two key respects: an estimate of π0 is employed, and the error rate of interest is E(V/R|R>0), the positive FDR (pFDR). Storey (2002) argues that pFDR is a more appropriate error rate to consider than the original FDR.

Any given rejection region (e.g., reject hypotheses for which the t-statistic has magnitude at least 3, or the raw p-value is at most 0.001) has an associated pFDR.

Indeed, in some reports, a fixed threshold such as 0.01 is chosen for the raw p-values, and the pFDR associated with that threshold is reported. But Storey (2002) argues that it is more appropriate to let the observed p-values determine the rejection region. To that end he proposes a new quantity, the q-value, as the basis for thresholding. The standard way to apply Storey's paradigm, and the one advocated in the neuroimaging context by Chen et al. (2009), is to reject those hypotheses with q-values below a specified level such as 0.05.

The q-value of an observed statistic. t is the min pFDR arising from a rejection region that contains t.

Under certain assumptions, most notably independence of the test statistics, the q-value can also be understood as the posterior probability that a null hypothesis is true, given a test statistic at least as extreme as the one oberseved. A key special case is when we take the observed raw p-values themselves as the test statistics: Storey (2002) then defines the q-value associated with p(i) (i=1,...,m) --- again, under the assumption that these p-values are independent --- as the smallest pFDR associated with some threshold  γ ≥ *p(i)*. 

ref: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3699340/



In this paper, we define the positive false discovery rate (pFDR) to be pFDR = E[V/R | R>0].

The term "positive" describes the fact that we have conditioned on at least one positive finding having occurred. 

Ref: 



pFDR阳性错误拒绝率，是基于至少拒绝一个H0的事实

pFDR的实际意义是，在pFDR错误控制下，当拒绝一个H0时，该假设为真实的概率

pFDR反应了在已经拒绝H0的情况下，H0=0的概率

可以认为pFDR时贝叶斯后验pvalue

 q-value量化了在观察统计量T = t时，拒绝H0所犯的最小pFDR



![img](https://img-blog.csdn.net/20150601134517559?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ2l0aHViXzI2NDU4NDkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)





如何理解和计算FDR

Ref: https://zhuanlan.zhihu.com/p/64925777





Ref: Statistical significance for genomewide studies

https://www.ncbi.nlm.nih.gov/pmc/articles/PMC170937/
