# Dante - Pro Labs

### WHAT IS DANTE

The ideal training environment for beginners!   
Dante is a modern and beginner-friendly Pro Lab that provides the opportunity to learn common penetration testing methodologies and gain familiarity with tools included in the Parrot OS Linux distribution.

<img src="https://user-images.githubusercontent.com/84217196/223658266-4c7b74cf-989a-4e0c-9fae-65627723fef9.png" width="850">

### Initial scan

Let’s scan the `10.10.110.0/24` subnet. We can initiate a ping sweep to identify active hosts before scanning them.  
`nmap -sn -T4 10.10.110.0/24 -oN active-hosts`  

![image](https://user-images.githubusercontent.com/84217196/223665467-e452edec-95f4-4d19-bfab-7683328ad9f1.png)
