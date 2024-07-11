# Seata-tutorial
### TCC mode: the correct way that should be applied.  

>**The first thing is TCC stand for Try-Comfirm-Cancel**
1. You will have a global transaction, it includes many branch transactions where each of them is located in a service in microservices. 
2. Each branch transaction will have a complete behavior like a RDBMS transaction: Try (write but not commit yet), Comfirm(commit), Cancel (rollback).
3. Try: check contraits in your bussiness logic, validate data, lock data or something else. After all, insert your data with state "TRY" (you can change the state name whatever you want). Other transaction shouldn't read this data as valid data.
4. Comfirm: when all the branch transaction complete their Try-phase without any problem. All branch transactions will go to 
Comfirm-phase, in this phase you should update state data in the Try-phase to "COMFIRM". Data is considered valid in this state and can be used in other bussiness logic.
5. Cancel: If there is any problem with Try-phase, the global transaction will be cancel. You should update state data in the Try-phase to "CANCEL" or delete it completely (be careful).

    
**NOTICE**:  
- When a global transaction begin, it must be started at a public function that mark @GlobalTransaction, and that function must be called from another Bean.  
  _e.g. beanA.call() -> beanB.excute(), NOT beanA.call() -> beanA.excute(). You can use debug to get more deeply dive._
- Notice your global transaction timeout, if it reach timeout, all the transaction branch will be cancel
- Comfirm and Cancel confirm phase is excute in another thread (acsyn), and will be retry until it success. You can create cronjobs to find there are some transaction stuck in Try state or check on Seata transaction monitor.


