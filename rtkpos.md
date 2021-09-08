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
int relposl(rtk_t *rtk, const obsd_t *obs, int nu, int nr, const nav_t *nav)
    /*-----------------------------------------------------------------------------
    rtk:rtk控制struct；
    obs:rover,base当前历元的所有观测量；
    nu,nr：rover、base各自的收星数；
    nav:导航数据； 
    *-----------------------------------------------------------------------------*/
```

程序执行流程：

1. 计算dt，龄期；
2. 初始化卫星列表rtk->ssat，为使能系统的每个卫星初始化sys,vsat(valid sat),snr;
3. satposs（ephemeris.c）计算卫星位置、钟差，输出rs, dts, var, svh；
4. zdres计算基站非差残差，输出y, e, azel；
5. ==intpref（没看）==
6. selsat 查找ref和user的共视卫星，返回ns, sat, iu, ir；
7. udstate

## zdres()

```c
static int zdres(int base, const obsd_t *obs, int n, const double *rs,
                 const double *dts, const int *svh, const nav_t *nav,
                 const double *rr, const prcopt_t *opt, int index, double *y,
                 double *e, double *azel) 
/* UD (undifferenced) phase/code residuals -----------------------------------
* calculate zero diff residuals [observed pseudorange - range] 
* output is in y[0:nu-1], only shared input with base is nav 
* args:  I   base:  0=rover,1=base
*        I   obs  = sat observations
*        I   n    = # of sats
*        I   rs [(0:2)+i*6]= sat position {x,y,z} (m)
*        I   dts[(0:1)+i*2]= sat clock {bias,drift} (s|s/s)
*        I   svh  = sat health flags
*        I   nav  = sat nav data
*        I   rr   = rcvr pos (x,y,z)
*        I   opt  = options
*        I   index: 0=rover,1=base
*        O   y[(0:1)+i*2] = zero diff residuals {phase,code} (m)
*        O   e    = line of sight unit vectors to sats
*        O   azel = [az, el] to sats
*---------------------------------------------------------------------------*/
```

输出y, e, azel 

y: undifferenced residual: obs measurement - geoditic distance - pcv 

## udstate()

```C
/* temporal update of states --------------------------------------------------*/
static void udstate(rtk_t *rtk, const obsd_t *obs, const int *sat,
                    const int *iu, const int *ir, int ns, const nav_t *nav)
```

执行流程：

1. udpos(rtk,tt);
   - 初始化状态向量x；
   - 构造状态转移矩阵F；
   - 构造状态转移过程中的噪声协方差阵Q（只包含了加速度部分）；
   - 执行卡尔曼滤波的时间更新过程；
2. ==电离层延迟参数的更新；（没看）==
3. ==对流层延迟参数的更新；（没看）==
4. ==GLONASS h/w bias 参数的更新；（没看）==
5. udbias(rtk,tt,obs,sat,iu,ir,ns,nav) 相位参数的更新（RTK肯定要用相位观测量）

