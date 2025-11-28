## **ACL (Access Control List) / Packet Filtering Protocol - L3**

- mechanism which allows or denies traffic based on a set of predefined rules (from one router to another)
    
- `show running-config | include access-list` or `show access-lists <acl_no./name>`
    
- **Working**
    
    - **Packet Filtering-** check each incoming/ outgoing packet against rules.
        
    - **Permit/ Deny-** Based on rules packets are either allowed/ denied
        
    - **Rule Order-** Rules are processed in sequential order. If a packet matches a rule, subsequent(below) rules are not evaluated.
        
    - **Implicit Deny-** At the end, there is an implicit "deny all" rule. (Every ACL needs permit at last to allow rest packets)
        
- <ins>**TYPES**</ins>
    
    - ### **Standard**
        
        - Divided into Named and Numbered
            
        - ACL no. range - ==**1-99**==
            
        - We can deny/ allow- <ins>network, host & subnet</ins>.
            
        - Filtering- **<ins>Source IP only</ins>**
            
        - ==Implemented closest to the Destination device.==
            
            - **Reason**\- if implemented closest to src- R needs to check src-ip matches dest-ip for all traffic | Else at dest it would only block if src matches (R **<ins>Processing overhead</ins>**)
        - **Standard Numbered ACL Config**
            
            - `access-list <ACL_no.> deny/permit <src_net-ID> <wild>`\- deny/permit entire network
                
                - or `access-list <ACL_no.> deny/permit <src_ip>`\- deny/permit 1 host
            - `access-list <acl no> permit any`
                
            - `int <int>`
                
            - `ip access-group <ACL_no.> in/out`
                
    - ### **Extended**
        
        - Divided into Named and Numbered
            
        - ACL no. range - ==**100-199**==
            
        - We can deny/ allow- <ins>network, host, subnet & FTP, HTTP, HTTPS</ins>
            
        - Filtering- <ins>**Source IP, Destination IP, Protocol, Port no.**</ins>
            
        - ==Implemented closest to the Source device.==
            
            - **Reason-** If implemented closest to dest- net traffic resources are consumed (**<ins>Bandwidth overhead</ins>**)
        - **Extended Numbered ACL Config**
            
            - `access-list <ACL_no.> deny/permit <protocol> host <src_ip> <wild> [operator] <src_port> host <dest_ip> <wild> [operator] <dest_port>` - don't give wildcard mask if specific host (host/any is option)
            - `access-list <ACL_no.> permit ip any any` - ip - protocol | any any - src & dest ip
            - `int <int>`
            - `ip access-group <ACL_no.> in/out`
        - **Operators**
            - **eq**: Matches exact port (e.g., `eq 80` matches TCP port 80, which is HTTP)
                
            - **lt**: Matches ports lower than a given number.
                
            - **gt**: Matches ports higher than a given number.
                
            - **range**: Matches a range of ports (e.g., `range 1000 2000` matches ports 1000-2000).
                
            - **neq**: Matches any port except the specified one.
                
- **Access List Rules**
    
    - Works in sequential order i.e-  All deny statements have to be given first
        
    - There should be at least one permit statement else all traffic blocked
        
    - Invisible statement- implicit "deny all" traffic by default.
        
    - Can have 1 access list per interface per direction- means 2 access lists per interface (inbound, outbound)
        
    - ==New entry in ACL will be placed at bottom of the list (could be manually added in Named ACL at any place \[but by default at bottom\])==
        
    - ==One line cannot be removed from Numbered ACL but we can remove it in Named ACL==
        
- <ins>To write ACL statement we have to decide</ins>
    
    - On which router to implement
        
    - Identify which is source and destination.
        
    - Which is ingress (packet going to router) & egress (packet leaving router) interface
        
- ### **Named ACL**
    
    - Access lists are identified using Names rather than numbers.
        
    - Names are case sensitive.
        
    - No limitation of numbers here.
        
    - One main advantage is <ins>editing of ACL is possible</ins> that is removing specific statement from the ACL is possible.
        
    - IOS versioṭn 11.2 or later allows named ACL.
        
    - **Standard Named ACL Config**
        
        - `ip access-list standard <name>`
            
        - `permit/deny host <src_ip> <wild>`
            
        - `permit any`
        - `int <int>`
            
        - `ip access-group <name> in/out`
            
        - <ins>**To add ACL in btw**</ins>\- `do sh access-list`, choose any no. btw 2 ACLs - `<btw-ACL_no.> deny/permit host <src-ip> <wild>`
        - **To delete-** `ip access-list standard <name>`, `no <ACL_no.>`
    - **Extended Named ACL Config**
        
        - `ip access-list extended<name>`
            
        - `` permit/deny <protocol> host <src_ip> <wild>`[operator] <src_port> host <dest_ip> <wild> [operator] <dest_port>` ``
            
        - `int <int>`
            
        - `ip access-group <name> in/out`
            
- ### **Numbered ACL**
    
    - Editing not possible, only last list can be
        
    - you can't insert a new rule in the middle of an existing list, and it will be placed at the bottom.
        
    - you cannot delete a single entry directly. You can only remove the entire ACL and recreate.
        
- `sh access-list`
    
    - **TCP**\- HTTP, Telnet, SSH, FTP, SMTP
    - **UDP**\- DNS, TFTP, DHCP, NTP
    - **ICMP**\- Ping, Traceroute
- <ins>**Practical**</ins>
- ![917222a3219ea195bb131ba257d7a748.png](../resources/917222a3219ea195bb131ba257d7a748.png)
    
    - **Std ACL (cond 1)**
        - **R1(numbered)**
            - ```bash
                access-list 1 deny 30.1.1.1
                access-list 1 permit any
                !
                int g0/0
                ip access-group 1 out
                ```
                
        - **R1(named)**
            - ```bash
                ip access-list standard cisco
                deny host 30.1.1.1
                permit any
                !
                int g0/0
                ip access-group cisco out
                ```
                
        - <ins>**Standard ACL Disadv**</ins>\- 30.1.1.1 can be blocked to ping 10.0.0.0/8 complete network | we cannot block it for only 10.1.1.1 (i.e. 1host- dest_ip)
    - **Ext ACL (cond 2)**
        - **R1(numbered)**
            - ```bash
                access-list 100 deny icmp host 10.1.1.1 host 30.1.1.2 echo 
                access-list 100 deny icmp host 10.1.1.1 host 30.1.1.2 echo-reply
                access-list 100 permit any any
                !
                int g0/0
                ip access-group 100 in
                ```
                
        - **R1(named)**
            
            
            
            - ```bash
                ip access-list extended cisco
                deny icmp host 10.1.1.1 host 30.1.1.2 echo
                deny icmp host 10.1.1.1 host 30.1.1.2 echo-reply
                permit ip any any
                !
                int g0/0
                ip access-group cisco in
                ```