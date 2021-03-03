<center style="font-size:50px; color:green">Procedure</center>

## 1，创建或替换

存储过程Procedure（P_PROCESS_RAPIDREPAIR_02）输入参数为PUSN         IN VARCHAR2,
                                                  PMO          IN VARCHAR2,
                                                  PUSERID      IN VARCHAR2,
                                                  PUPN         IN VARCHAR2,
                                                  PMODEL       IN VARCHAR2,
                                                  PMODELFAMILY IN VARCHAR2,
                                                  PPRESTAGE    IN VARCHAR2,
                                                  PSTAGE       IN VARCHAR2,
                                                  PLINE        IN VARCHAR2,
                                                  PMOTYPE      IN VARCHAR2,
                                                  PWORKSTATION IN VARCHAR2,
                                                  PERRORCODE   IN VARCHAR2,
                                                  PREPAIRTYPE  IN VARCHAR2,
                                                  PRETURNSTAGE IN VARCHAR2,
                                                  PREPAIRID    IN VARCHAR2,
                                                  PGRDFLAG     IN VARCHAR2,
                                                  PPANELID     IN VARCHAR2,
                                                  PTOUR        IN VARCHAR2,
                                                  PSPECERRFLAG IN VARCHAR2

## 2，查询

SFCUSNITEM和SFCUSNITEM_BK条件usn=输入参数PUSN     and   CATEGORY='LK'  

## 3，如果

输入参数PSPECERRFLAG  的值为 'Y' ，

遍历2中查询的数据  

- 1.Dismantle   修改SFCUSNITEM表，修改SFCCSNUSED表，插入SFCUSNITEM_BK表  
- 2.Record    插入SFCRAPIDTRANSACTION表   

结束遍历

- 3.defect information    插入SFCUSNDEFECT
- 4.repair information     插入SFCUSNREPAIR
- 5.1.transaction defect information     插入SFCTRANSACTION
- 5.2.transaction repair information   插入SFCTRANSACTION
- 6.return stage      修改SFCUSN
- 7.Grade:NN     IF PGRDFLAG='Y'    插入SFCUSNGRADE



如果输入参数PSPECERRFLAG  的值不为 'Y' ，

遍历2中查询的数据  

- --1.Dismantle    修改SFCUSNITEM表，修改SFCCSNUSED表，插入SFCUSNITEM_BK表  
- --2.Record   插入SFCRAPIDTRANSACTION表   

结束遍历

- 3.return stage      修改SFCUSN表

## 4，出现异常

ROLLBACK

最后END

