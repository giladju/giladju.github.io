---
layout: post
title:  "Devops Shifting Sands"
date:   2017-02-22 11:14:49 +0200
categories: devops tips 
---

I’ve been in IT/system administration since I started working after completing my B.Sc., mostly unofficially while filling the capacity of QA manager, and recently doing what is now called DevOps, all in all over 21 years of experience, in setting up, deploying, testing and troubleshooting computer systems. 

And still it surprises me when I interact with my customers when  their attitude towards the DevOps deliverables is as if it’s the simple part of the system, where as all the smart stuff developed by the good people coding the Intellectual Property of the company, that is where the complex stuff is done, that is where bugs are tolerable, whereas the underlying system must work 100% 24/7. Anything less reflects badly on the DevOps engineers’ performance.

Underestimating the effort in setting up the company infrastructure, that might stem from the hype around virtual services and “the cloud”, causes a two pronged risk to the development organization, in addition to the infrastructure simply not working up to par, the people involved in maintaining said system get frustrated with the attitude of them being the “help” as opposed to high self-esteem one gets when working in a highly sought out discipline.

Yes, there is an issue with hiring good people in a much needed area of expertise, and the truth is that DevOps personal supply a service to the organization, mainly to the development team. It’s best that the DevOps engineers acknowledge this. On the other hand managers need to adapt to managing DevOps engineers. There seems to me to be a lack of understanding of the DevOps work, the work of maintaining uptime of a system while the sands constantly shift below and within the system.

The idea that a small team is all that is needed to maintain a multitude of environments (Standalone, Development, QA, Staging, CI, Production) while that 3rd party software shifts and changes, and development code requirements get altered without notice, might hold some water in a very small team, but everything becomes a lot more complex when the team grows. The need for Ad Hoc environments grow exponentially. The code involved in setting up the system becomes in project in itself that requires Continuous Integration and Testing, and breaks more often than you’d think. And here lies the issue, development managers live under the assumption that coding the infrastructure, be it in Ansible, Chef or Puppet, is 100% bullet proof. Linux is an OS, it does not change, Mongo is a database, it does not change, Apache is a web service, it does not change - and none of the above have bugs.

I got a painful reminder of the shifting sands, in a recent two day project at a customer. The customer needed to upgrade the hardware of a five year old setup of Bugzilla and SVN, that was working perfectly for them on RedHat 4.x. The new system was setup on a CentOS 7.3 server and after getting some help from a colleague that set this up originally 

