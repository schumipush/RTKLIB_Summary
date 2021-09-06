[toc]

## rtkpos()
```c
int rtkpos(rtk_t *rtk, const obsd_t *obs, int n, const nav_t *nav)
    /*-----------------------------------------------------------------------------
    rtk:rtk控制struct；
    obs:rover,base当前历元的所有观测量；
    n：观测量总数，也即rover、base总的收星数；
    nav:导航数据； 
    *-----------------------------------------------------------------------------*/
```

函数内流程：

1. 设置基站坐标
2. 统计rover、base各自的卫星数nu、nr；
3. 计算rover坐标（pntpos）
4. 计算tt：当前历元与上一历元的时间差；
5. 模式为单点定位时的对应操作；
6. 模式为PPP时的对应操作
7. 模式为移动基线时的对应操作
8. 模式为其他(rtk)时的对应操作
   - 计算龄期：rtk->sol.age，rover,base之间的接收到信号的时间差；
   - 判断龄期是否过大；
9. relpos(rtk,obs,nu,nr,nav);
10. outsolstat(rtk,nav);





## relpos()

```c
int relpos_original(rtk_t *rtk, const obsd_t *obs, int nu, int nr, const nav_t *nav)
    /*-----------------------------------------------------------------------------
    rtk:rtk控制struct；
    obs:rover,base当前历元的所有观测量；
    nu,nr：rover、base各自的收星数；
    nav:导航数据； 
    *-----------------------------------------------------------------------------*/
```

程序执行流程：

1. 计算dt，龄期；
2. 

