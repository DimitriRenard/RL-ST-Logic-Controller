# PLC code extraction
- Initial step actualization:
```
#task[1].IS := #X10;
#task[2].IS := #X20;
#task[3].IS := #X30;
#task[4].IS := #X40;
#task[5].IS := #X50;
#task[6].IS := #X60;
#task[7].IS := #X70;
#task[8].IS := #X80;
#task[9].IS := #X90;
#task[10].IS := #X100;
#task[11].IS := #X110;
#task[12].IS := #X120;
#task[13].IS := #X130;
```
- Initial condition actualization:
```
#task[1].IC := NOT "wait1";
#task[2].IC := NOT "wait2";
#task[3].IC := (#position = 0) AND #retracted AND "wait1" AND #up AND NOT "detected";
#task[4].IC := #up AND #retracted AND "detected" AND (#position = 1);
#task[5].IC := #up AND #retracted AND "detected" AND (#position = 2);
#task[6].IC := (#position = 0) AND #retracted AND "wait2" AND #up AND NOT "detected";
#task[7].IC := #up AND #retracted AND "detected" AND (#position = 3);
#task[8].IC := #up AND #retracted AND NOT "detected" AND (#position = 2);
#task[9].IC := "p_exit";
#task[10].IC := (#position = 2) AND #retracted AND #up AND NOT "detected" AND "wait1";
#task[11].IC := (#position = 2) AND #retracted AND #up AND NOT "detected" AND "wait2";
#task[12].IC := #retracted AND "wait1" AND #up AND NOT "detected" AND (#position = 1);
#task[13].IC := #retracted AND "wait2" AND #up AND NOT "detected" AND (#position = 3);
```
- State vector creation:
```
#State := 0;
FOR #i := 0 TO 13 DO
    #State := #State + BOOL_TO_INT(#task[13-#i].IS) * 2 ** (#i * 2);
    #State := #State + BOOL_TO_INT(#task[13-#i].IC) * 2 ** ((#i * 2) + 1);
END_FOR;
```
- State lookup in the table:
```
#min := 0;
#max:=UPPER_BOUND(ARR := #ArEtat, DIM := 1);

WHILE #min < #max DO
    #mid := (#min + #max) / 2;
    IF  #ArEtat[#mid] = #S_State THEN
        #Action := #ArAction[#mid];
        RETURN
        ;
    ELSIF  #ArEtat[#mid] < #S_State THEN
        #min := #mid + 1;
    ELSE
        #max := #mid - 1;
    END_IF;
    
END_WHILE;

IF #ArEtat[#min] = #S_State THEN
    #Action := #ArAction[#min];
    RETURN;
ELSE
    #Action := 0;
    RETURN;
END_IF;
```
- Authorize the task corresponding to the state:
```
FOR #i := 0 TO 13 DO
    #task[#i].AUTf := 0;
END_FOR;
#task[#ActionFound].AUTf := 1;
```
- Logic filter:
```
#task[1].AUT := #task[12].IS AND #task[1].AUTf;
#task[2].AUT := #task[13].IS AND #task[2].AUTf;
#task[3].AUT := #task[6].IS AND #task[3].AUTf;
#task[4].AUT := #task[12].IS AND #task[4].AUTf;
#task[5].AUT := #task[8].IS AND #task[9].IS AND #task[10].IS AND #task[11].IS AND #task[5].AUTf;
#task[6].AUT := #task[3].IS AND #task[6].AUTf;
#task[7].AUT := #task[8].IS AND #task[13].IS AND #task[7].AUTf;
#task[8].AUT := #task[5].IS AND #task[7].IS AND #task[10].IS AND #task[11].IS AND #task[8].AUTf;
#task[9].AUT := #task[5].IS AND #task[9].AUTf;
#task[10].AUT := #task[5].IS AND #task[8].IS AND #task[11].IS AND #task[10].AUTf;
#task[11].AUT := #task[5].IS AND #task[8].IS AND #task[10].IS AND #task[11].AUTf;
#task[12].AUT := #task[1].IS AND #task[4].IS AND #task[8].IS AND #task[12].AUTf;
#task[13].AUT := #task[2].IS AND #task[7].IS AND #task[8].IS AND #task[13].AUTf;
```
- Task programs example:
```
(*Task 1 : GConv1_T1*)
#FTX10 := #task[1].AUT;
#FTX11 := #X11 AND "wait1";

#X10 := #FTX11 OR #X10 AND NOT #FTX10 OR "init";
#X11 := (#FTX10 OR #X11 AND NOT #FTX11) AND NOT "init";
```
