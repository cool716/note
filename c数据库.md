<center style="font-size:50px;color:green">数据库</center>

一，Sql  Server

```
Server：10.42.21.38
DB：WKSPSDC
賬號：WKSPSDC
密碼：WKSp0rtal11 
```

人员信息： 

D-11  D-12 A-11 A-12     是   **DL** 

A-20 C-20 是     **IDL**

B-20是   **臺派** 

==查询WOK所有IDL==

```
select *  from  WKSPSDC.dbo.PS_Z_EE1_CN1_VW  
where Z_SITE_ID = 'WOK'   and	EMPL_CLASS in('A','C') and  Z_LABOR_TYPE='20'
```

