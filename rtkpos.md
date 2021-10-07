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
7. udstate 执行时间更新过程；
8. 初始化x, xa, Pp;
9. 循环（正常情况执行一次循环体，当moveb模式时循环3次）：
   - zdres 计算rover的非差残差，输出y, e, azel；
   - ddres 计算双差残差

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
   - 执行卡尔曼滤波的时间更新过程，输出rtk->x, rtk->P；
2. ==电离层延迟参数的更新；（没看）==
3. ==对流层延迟参数的更新；（没看）==
4. ==GLONASS h/w bias 参数的更新；（没看）==
5. udbias(rtk,tt,obs,sat,iu,ir,ns,nav) 状态向量中相位参数的更新（RTK肯定要用相位观测量）
   - 相位部分设置为站间单差（这样就可以把参考卫星的选择留到ddres()中）；

## ddres()

```C
static int ddres(rtk_t *rtk, const nav_t *nav, const obsd_t *obs, double dt, const double *x,
                 const double *P, const int *sat, double *y, double *e,
                 double *azel, const int *iu, const int *ir, int ns, double *v,
                 double *H, double *R, int *vflg)
/* double-differenced residuals and partial derivatives  ----------------------
*        O rtk->ssat[i].resp[j] = residual pseudorange error
*        O rtk->ssat[i].resc[j] = residual carrier phase error
*        I rtk->rb= base location
*        I nav  = sat nav data
*        I dt = time diff between base and rover observations (usually 0)
*        I x = rover pos & vel and sat phase biases (float solution)
*        I P = error covariance matrix of float states
*        I sat = list of common sats
*        I y = zero diff residuals (code and phase, base and rover)
*        I e = line of sight unit vectors to sats
*        I azel = [az, el] to sats
*        I iu,ir = user and ref indices to sats
*        I ns = # of sats
*        O v = double diff innovations (measurement-model) (phase and code)
*        O H = linearized translation from innovations to states (az/el to sats)
*              (partial derivatives)
*        O R = measurement error covariances
*              (double diff measurement error covariances)
*        O vflg = bit encoded list of sats used for each double diff  
*----------------------------------------------------------------------------*/
```

函数执行流程：

1. baseline() 计算基线向量和基线长；
2. 计算ion、trop延迟因子；
3. 外、内两层循环：1）遍历系统，2）遍历观测量类型（L1相位、L2相位、L1伪距、L2伪距）
   - 先参考卫星，即仰角最高的卫星，得到i是该卫星在共视卫星中的序号；
   - v是 相位/伪距 误差的测量值与预测值的差：

