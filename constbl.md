[toc]

# rnx2rtkp_main.c文件
- 命令行参数：
  - 1）rover观测量文件，2）base观测量文件，3）至少一个导航文件；
- 

# rtkpos.c文件
## ddres()函数



## constbl()函数

1. 位置：rtkpos.c
2. 定义
   - `static int constbl(rtk_t *rtk, const double *x, const double *P, double *v,
                        double *H, double *Ri, double *Rj, int index)` 
3. 调用
   - `if (opt->mode==PMODE_MOVEB&&codnstbl(rtk,x,P,v,H,Ri,Rj,nv)) {
             vflg[nv++]=3<<4;
             nb[b++]++;
         }`
4. 





