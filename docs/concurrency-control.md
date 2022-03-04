## Read Write Anomalies

dirty read
-----------------------
    T1    |    T2
    W(A)  | 
          |    R(A)
    Abort |    
          |    COMMIT
-----------------------

non-repeatable read
-----------------------
    T1     |    T2
    R(A)   | 
           |    W(A)
           |    COMMIT
    R(A)   |    
    COMMIT |
-----------------------

phantom read
-----------------------
    T1     |    T2
    R(A)   | 
    R(B)   |    
           |    W(A)
           |    COMMIT
    R(A)   |
    R(B)   | 
-----------------------

lost update
-----------------------
    T1     |    T2
    R(A)   | 
           |    R(A)   
    W(A)   | 
           |    W(A)
    COMMIT |
           |    COMMIT
-----------------------

dirty write
-----------------------
    T1    |    T2
    W(A)  | 
          |    R(A)
          |    A := A + 5
    Abort |    
          |    COMMIT
-----------------------

write skew
A = 100
B = 150
-------------------------------------
    T1            |    T2
    R(A)          | 
    R(B)          |    
                  |    R(A)
                  |    R(B)
    IF A + B >= 0 |
    THEN A -= 200 |
    COMMIT        |
                  |    IF A + B >= 0
                  |    THEN B -= 200
                  |    COMMIT
-------------------------------------

## OCC
R <- read phase start time
V <- validation phase start time
W <- write phase end time
-----------------------
    T1    |    T2
          | 
    V-----|    
          |    
          |-----V
          |
          |    
-----------------------

## MVCC 
### ex1
TS(T1) = 1
TS(T2) = 2
-----------------------
    T1            |    T2
    BEGIN         | 
    R(A)          |
                  |    BEGIN
                  |    W(A)
    R(A) <- 123   |
    W(A)          |
    COMMIT        |    
                  |    COMMIT
-----------------------

Datbase
|---version---|---value---|---begin---|---end---|
|     A0      |     123   |   0       |    2    |
|     A1      |     456   |   2       |    /    |
|     A2      |     133   |   1       |    /    |

Txn Status Table
|---TxnID---|---Timestamp---|---Status---|
|   T1      |      1        |  COMMIT    |
|   T2      |      2        |  COMMIT    |

### ex2
TS(T1) = 1
TS(T2) = 2
-----------------------
    T1     |    T2
    BEGIN  | 
    R(A)   |
    W(A)   |
           |    BEGIN
           |    R(A) <- 123
           |    W(A) (wait)
    R(A)   |
    COMMIT |    
           |    COMMIT
-----------------------

Datbase
|---version---|---value---|---begin---|---end---|
|     A0      |     123   |   0       |    1    |
|     A1      |     456   |   1       |    2    |

Txn Status Table
|---TxnID---|---Timestamp---|---Status---|
|   T1      |      1        |  COMMIT    |
|   T2      |      2        |  COMMIT    |


## 2PC
### Non 2PC
---------------------------------
    T1            |    T2
    BEGIN         |    BEGIN
    Lock(A)       |
    R(A)          |
    A = A - 100   |    
    W(A)          |
    UNLOCK(A)    |
                  |    Lock(A)
                  |    R(A)
                  |    UNLOCK(A)
                  |    LOCK(B)
                  |    R(B)
                  |    UNLOCK(B)
                  |    ECHO A + B
                  |    COMMIT
    LOCK(B)       |
    R(B)          |
    B = B + 100   |
    W(B)          |
    UNLOCK(B)     |
    COMMIT        |
-----------------------------------

Initial state: A = 1000, B=1000
T2 Output: 1900

### 2PL
---------------------------------
    T1            |    T2
    BEGIN         |    BEGIN
    Lock(A)       |
    R(A)          |
    A = A - 100   |    
    W(A)          |
    LOCK(B)       |
                  |    Lock(A)
                  |    R(A)
                  |    UNLOCK(A)
                  |    LOCK(B) <-------- wait
                  |    R(B)
                  |    UNLOCK(B)
                  |    ECHO A + B
                  |    COMMIT
    R(B)          |
    B = B + 100   |
    W(B)          |
    UNLOCK(A)     |
    UNLOCK(B)     |
    COMMIT        |
-----------------------------------

Initial state: A = 1000, B = 1000
T2 output: 2000

## Deadlock
------------------------------
    T1         |    T2
    Lock(A)    |
    W(A)       |
               |    Lock(B) 
               |    W(B)
               |    Lock(A)
    Lock(B)    |
    R(B)       |
------------------------------