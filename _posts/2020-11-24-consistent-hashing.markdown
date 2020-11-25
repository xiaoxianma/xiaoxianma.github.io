---
layout:     post
title:      "Consistent Hashing"
subtitle:   ""
date:       2020-11-24 22:01:00
author:     "Xiaoxianma"
header-img: "img/home-bg-oakley-on-keyboard.jpg"
tags:
    - system_design
    - sharding
    - database
    - shashing
    - scalability
---

## Background
To make database scalable and high available, we only need to do two things sharding and replica. We will talk about database sharding in this post. There are two shardings vertical sharding and horizontal sharding. Vertical sharding means sharding tables into different hosts or sharding one table into several tables by columns. Horizontable sharding means sharing table by rows.

## An intuitive solution
Initially, we have 10 hosts storing our data. Data can be stored by `%10`. Someone we add one more host to 11 hosts in total. Now we run into a problem as almost all data have to be migrated by `%11` to hosts. There are three main issues:  
1. Data migration process slow.  
2. During data migration, servers are getting stressed and may be down.  
3. Large data migration may run into data inconsistency.  

## Consistent Hashing
![](/img/posts/consistent-hashing.png)

### Simple solution  
![](/img/posts/consistent-hashing-simple-1.png)
![](/img/posts/consistent-hashing-simple-2.png)

There are still some problems:
- One server is added. The neighbor hosts(one or two) are getting stressed and may impact normal serving.  
- Everytime, data from new server is only coming from surroudning 1-2 servers. In this case, data are not distributed equially.  

### Practical solution
![](/img/posts/consistent-hashing-practical-solution.png)
