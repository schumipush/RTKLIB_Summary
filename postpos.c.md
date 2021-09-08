[toc]

# postpos.c中的函数

## procpos()：进行rtk初始化，调用rtkpos，打印.pos文件

```C
static void procpos_original(FILE *fp, const prcopt_t *popt, const solopt_t *sopt, rtk_t *rtk, int mode)
```

程序执行流程：

1. 确定solstatic（是否静态定位？若是，仅输出最后一个定位结果到.pos文件）；
2. 满足一定条件时执行rtkinit。条件是：1)forward || 2)backward || 3)combined中的forward过程 || 4)AR模式为fix and hold;
3. 循环：每次读取一个历元的观测量，并在循环体中解算：
   - 剔除卫星 / 选择参与定位的卫星，若属于参与定位的系统且没有被预先要求剔除，则选取备用；
   - ==/* carrier-phase bias correction */：corr_phase_bias_fcb / corr_phase_bias_ssr==
   - rtkpos(rtk,obs,n,&navs)；
   - 打印.pos文件；
4. 解算完全部历元，且模式为静态时，打印最后一个定位结果到.pos文件；
5. end；





