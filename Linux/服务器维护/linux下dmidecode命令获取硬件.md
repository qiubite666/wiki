1、查看内存槽数、那个槽位插了内存，大小是多少

> dmidecode|grep -P -A5 "Memory\s+Device"|grep Size|grep -v Range

2、查看最大支持内存数

> dmidecode|grep -P 'Maximum\s+Capacity'

3、查看槽位上内存的速率，没插就是unknown。

> dmidecode|grep -A16 "Memory Device"|grep 'Speed'

4、查询电源功率

> dmidecode |grep "System Power Supply" -A 16

