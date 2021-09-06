[toc]

# rtkcmn中的函数

> time相关

- double  str2num(const char *s, int i, int n);
- int     str2time(const char *s, int i, int n, gtime_t *t);
- void    time2str(gtime_t t, char *str, int n);
- gtime_t epoch2time(const double *ep);
- void    time2epoch(gtime_t t, double *ep);
- gtime_t gpst2time(int week, double sec);
- double  time2gpst(gtime_t t, int *week);
- gtime_t gst2time(int week, double sec);
- double  time2gst(gtime_t t, int *week);
- gtime_t bdt2time(int week, double sec);
- double  time2bdt(gtime_t t, int *week);
- char    *time_str(gtime_t t, int n);

- gtime_t timeadd  (gtime_t t, double sec);
- double  timediff (gtime_t t1, gtime_t t2);
- gtime_t gpst2utc (gtime_t t);
## utc2gpst() 从utc中减去周跳
gtime_t utc2gpst(gtime_t t) 
	- utc to gpstime --------------------------------------------------------------
	- convert utc to gpstime considering leap seconds
	- args   : gtime_t t        I   time expressed in utc
	- return : time expressed in gpstime
	- notes  : ignore slight time offset under 100 ns

- gtime_t gpst2bdt (gtime_t t);
- gtime_t bdt2gpst (gtime_t t);
- gtime_t timeget  (void);
- void    timeset  (gtime_t t);
- double  time2doy (gtime_t t);
- double  utc2gmst (gtime_t t, double ut1_utc);
- int read_leaps(const char *file);

> trace相关函数

	- void traceopen(const char *file);
	- void traceclose(void);
	- void tracelevel(int level);
	- void trace    (int level, const char *format, ...);
	- void tracet   (int level, const char *format, ...);
	- void tracemat (int level, const double *A, int n, int m, int p, int q);

## traceobs
	- void traceobs (int level, const obsd_t *obs, int n);
rtkpos()中，trace等级为4；

	- void tracenav (int level, const nav_t *nav);
	- void tracegnav(int level, const nav_t *nav);
	- void tracehnav(int level, const nav_t *nav);
	- void tracepeph(int level, const nav_t *nav);
	- void tracepclk(int level, const nav_t *nav);
	- void traceb   (int level, const unsigned char *p, int n);


