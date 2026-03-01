### Lab: Username enumeration via different responses

## 1 Vulnerability

**Like desciption of the lab, we need to brute-force attack**

**Use burp and add § to attack like sql injection lab**

## 2 Exploitation

**In this lab type, I take two wordlists about username and password. I login random account then accessing history proxy login to attack**


**In this lab, we can use Cluster bomb attack and find username and password at the same time. However, it is very slow and consums a large amount of resources, so I use sniper attack**


**Firstly, I add § into username and attack with range list I take from username wordlist**



**After finding username, I replace username on board position. Then I attack to find password**


**Account found successfully**

