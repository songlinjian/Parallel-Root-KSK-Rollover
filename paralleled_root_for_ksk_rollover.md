# Architectural reconsideration on ICANN's Root Zone KSK rollover

## Introduction

DNSSEC is a technology to keep origin authentication and data integrity of DNS. There is a consensus in the technical community that that maintaining an up-to-date DNSSEC Root Zone KSK is essential to ensuring DNSSEC-validating DNS resolvers continue to function following the rollover. ICANN as IANA Functions Operator are responsible for the Root Zone KSK rollover. For more information please visit [www.icann.org/resources/pages/ksk-rollover][1].

At the time of writing, the Root Zone KSK is expected to undergo a carefully-organized rollover as described in [2017 KSK Rollover Operational Implementation Plan][2]. However, the [process is postponed][3] on 27 September 2017 due to some recently obtained data showing significant number of resolvers used by Internet Service Providers (ISPs) and Network Operators are not yet ready for the Key Rollover. Further details on [the data and reasoning behind the postponement][13] were given afterwards.

Note that the recent update on the Root KSK Rollover Project shows that ICANN is calling for feedback and input from the community on generally "how & when to proceed", more specific, the objective criteria and metric to indicate right time to roll the key.

This paper review the problem, analysis the rolling model of KSK, and propose a new architecture as a complement or preparation to the current [ICANN's KSK rollover implementation plan][2]. It is proposed to build a paralleled root system for RFC5011-rollover which divides the ready and unready resolvers into two groups. In one hand, the obsolete resolvers can still survive after KSK rollover in the new root system. On another hand, the ready resolver can go through the rollover without impact. It ends up with a win-win situation.

## Problem Analysis

According to the specification, [RFC5011][4] defines a means for automated, authenticated, and authorized updating of DNSSEC "trust anchors". The essence of this method is the effective protection against the threat model called "N-1 key compromises of N keys" by setting proper hold-down timers in resolver. As a IETF Standards Track document, RFC5011 is chosen as the de facto solution in [SSAC Advisory on DNSSEC Key Rollover in the Root Zone][8],  ICANN's [Root Zone KSK Rollover Plan][5] and the final [operational implementation plan][2]. The assumption is that any resolver who support this protocol can its trust anchors with a new SEP key (KSK). 

Although a carefully-organized rollover is designed on the authoritative side, the most issues raised are on the resolver side: not all the resolvers are ready for the Root zone KSK rollover. To simplify the problem, according to the readiness of RFC5011 and the operational status, resolvers are categorized into three types (Note that the resolver in this context is validating resolver): 

* Dying Resolver (DR): obsolete resolvers and DNS modules which is deployed in the network without careful maintenance. They have irresponsible operators who have no clue of RFC5011 stuff, the new key, revoke bit and hold-on timers. For example DNS modules in IoT gateway or devices. Those resolvers will definitely fail(die) after the rollover. This type is characterized that fix will be done only after the serious failure. BTW even there is no such KSK rollover, these resolver with obsolete TA will die eventually encounter a broken chain of trust once the TA is compromised due to various reason.

* Curable Resolver (CR): normal resolvers which can not roll the new key due to misconfiguration, software implementation bugs and network situation (IPv6 fragmentation). It is very complex to dig out why this happen, and it is very hard fixed the problem right away. Their operators may be informed and educated later by continuous report and advocacy. Or the system are updated due to companies' regular check and updates. Anyway this type of resolvers is characterized that all of them can be fixed as the time goes by.

* Healthy Resolver (HR): Good resolvers support RFC5011 and their operators are very active and alert on the security updates and new DNS extensions.

The classification is intuitive, but the question is what's the proportions of each type of resolvers in the world. The more difficult and more desirable question is what's the proportions of end users behind the three types of resolver. The answers of these question will help people to make reasonable decisions. As a background, in [SAC063 Recommendation-03][8], a clear and objective metrics is recommended for acceptable levels of “breakage” resulting from a key rollover. If an reasonable boundary is broken during the rollover, the decision to rollback the rollover can be made quickly and efficiently.

There are some proposals aiming to answer these questions. RFC8145 is a DNS extension enabling resolver to tell the authoritative server its current trust anchor. The ICANN's decision of postponing is based on the data got from the new function in September 2017. The ksk-sentinel draft is a way to figure out the second question: what's the portion of end users under the unready resolvers. This draft enable the end users to determine (then report) the trusted key state of the resolvers they are using.

One problem on these data analysis is how accurate is this data. Sometimes data may be messed by factors of DNS implementation behaviors as reported in [September-2017 information][9].  Sometimes the data may have no statistical significance on behalf of all resolvers around the world. It is true that the new data are collected with the cooperation with entities which update the newest extension of RFC8145 and ksk-sentinel draft in future. However, even all the data is accurate, how can people make a decision to roll the key if the data indicate a large number of resolvers and end users will be affected during these process. What is the number? What is the boundary? Sadly they are unknown yet to the author's best knowledge.

## Is there any alternative way? Learn from old lessons

In ICANN's KSK rollover documents and discussions, people seems ready for some risks and failures due to the rolling. It is true that Internet has built-in resilience in many layers. People can always fixed the problem when failures come. But did people explore all the possibility to solve the problem? Is there any alternative way? To understand the problem more deeply and find a way out, there should be a new perspective and analysis on the situation.

Obviously the precondition of using RFC5011 is broken in current situation that not all resolvers are ready to roll the key. The portion of unready resolver and users is unknown and they may be not a trivial part. If we treat the KSK-2017 DNSSEC as an advanced technology compared with KSK-2010 DNSSEC. We are facing the problem when to turn off KSK-2010 DNSSEC system. In another word, it is not the problem on how to automatically and safely roll the key. It is the problem of new technology adoption and transition with less suffering as possible. 

Sounds familiar? Yes, people has long-term experience and wisdom in this field, like DNSSEC, HTTPs, and IPv6 adoption. There are two critical factors for the success of a new technology adoption and transition. 

* One is the incentive of change. That is to say people can benefit by adopting this new technology. It is best that the technology has "the day you deploy, they day you gain" model. Luckily the users and validating resolvers already has incentive to deploy DNSSEC so they surely accept to adopt KSK-2017 DNSSEC as soon as possible.

* The other one is that people can opt in and opt out based on they careful decision. It means there should be a backward compatibility in design for new technology as well as the capability of incremental adoption. The success of Internet itself is benefit from this important feature forming an network of networks. Currently there is no such design in adoption of KSK-2017 DNSSEC. It is vary sad!

Due to the lack of incremental adoption consideration, the plan of Root Zone KSK Rollover is act like a switch. People now are anxious standing by the switch handle waiting for a decision to turn off KSK-2010 DNSSEC system and launch KSK-2017 DNSSEC system. You can imagine the similar situation that one day you were told that IPv4 will be turn off in one month time. It sounds like a rude and hard warning for preparation of the switch. 

One lesson from IPv6 adoption and transition is that there is a dual-stack period in which IPv4 and IPv6 network are both work. The lesson from DNSSEC and HTTPs is that the server support both old tech and the new one. In this adoption and transition model, the decision of turning off old technology system can be made based on clear and objective metrics from the adoption of new technology and proportion of users rely on old system. 

## Concept and Transition Modeling 

Based on the analysis and lessons we learned, the intuitive answer to the KSK-rollover question is setting up a system (or two systems) that running in a dual-stack model in which the whole system can both speak KSK-2010 and KSK-2017 DNSSEC. In this scenario, the healthy resolvers who support RFC5011 can adopt the KSK-2017 DNSSEC. The unready resolvers including the dying and curable resolvers can still survive relying root zone update of KSK-2010 DNSSEC. 

       +----------+                 +----------+
       | KSK-2010 |                 | KSK-2017 |
       | Root     |                 | Root     |
       | System   |                 | System   |
       +----------+                 +----------+
            ||                           ||
            ||                           ||
            ||                           ||
            v                             v
        +----------+                 +---------+
        | Dying and|                 | Healthy |
        | Curable  |                 | Resolver|
        | Resolver |                 |         |
        +----------+                 +---------+
Figure 1 The concept model of two Root system in parallel

Figure 1 depicts a simple concept model of New KSK rollover with two root servers working in parallel.They are defined as follows: 

* KSK-2010 Root System and its resolvers. KSK-2010 Root system is for obsolete resolver (DR and CR) who can support the communication and validating with KSK-2010.  

* KSK-2017 Root system and its resolvers. KSK-2017 Root System is for Healthy resolver who successfully roll their key to KSK-2017 or ready for RFC5011 rollover.

This model is clear and desirable for user to opt in. But it is a question how to start from current root system Now in Phase D of ICANN's Root Zone KSK Rollover Plan.


### Transition from current root system to paralleled root system

                   +                                 ( KSK+2017)
                   |                                   ROLLING
     +---------+   |    +---------+           +----------+
     | Current |   |    | KSK+2010|           | RFC5011  +--+
     | Root    |   |    | Root    |           |Ready Root|  |
     | System  |   |    | System  |           | System   +^-+
     +---------+   |    +---------+           +----------+
          ||       |         ||                    ||
          ||    -------->    ||                    ||
          ||    -------->    ||                    ||
           V   TRANSITION    V                      V
     +---------+   |    +----------+           +--------+
     |   All   |   |    |Dying and |           |Healthy |
     |resolves |   |    |Curable   | +-------> |Resolver|
     |         |   |    |Resolver  | TRANSFORM |        |
     +---------+   |    +----------+           +--------+
                   |
                   +
Figure 2 the concept model of Root system transition

Regarding the transition from current root system to the two paralleled root systems, one approach is to setup a new set of root system (consists of another 13 servers for example), and switch the Healthy Resolver via a "TRANSFROM" process from current root system to a newly setup root system.

In figure 2 the KSK-2010 Root System consists of exactly current root server which is to be updated to support a new DNSSEC extension to support TRANSFORM process defined in next section. RFC5011 Ready Root System in figure 2 is a paralleled Root system which consists of a new set of root servers hosting root zone containing the similar root zone of KSK-2010 Root System except for the apex NS records.

Currently all resolver are "connected" with the KSK-2010 Root System. Healthy Resolver is expected to switch from existing root servers to the paralleled root servers. This process is called TRANSFORM. The Health resolver is the targeted group of resolvers in ICANN's KSK rollover plan.

In this model KSK-2017 rollover can be triggered without update of KSK-2010 Root system. There is no impact on the existing Dying Resolver and Curable Resolver no matter what's proportion they are. This model enable incremental deployment in the design.

The question on when to turn off the KSK-2010 Root System is depended on the proportion of DR and CR. If the number is under some boundary say 0.5%, then reasonable decision can be made. In addition, this model can safely distinguish the HR from DN and CR which helps people to located the source of DR and CR, and accelerate the process of TRANSFORM.

### Possible Time line for Phased New KSK rollover

According to the description of new model for KSK rollover, the whole process may consist of several phases with a time line.

* 1) Phase A: setup a paralleled root system similar with current root system with different NS servers. It is prepared for RFC5011 Ready Root System.

* 2) Phase B: develop the specification of DNS extension for TRANSFORM function. Develop and distribute DNS extension patch to root servers and available resolvers. 

* 3) Phase C：promotion work are necessary including speech on various meetings, conferences, workshops to outreach.  

* 4) Phase D: trigger the initial TRANSFORM on KSK-2010 Root Server

* 5) Phase E：trigger the KSK-2017 rollover process in RFC5011 Ready Root Server when the proportion is beyond a certain number(say 80%). Future KSK rollover after KSK-2017 can be conducted every 5 years independent of the status of KSK-2010 Root Server 

* 6) Phase F: If the proportion of DR and CR is under a certain boundary(say 0.5%), turn off the KSK-2010 Root Server. Phase F may be never come or long time to come.

Note that Phases described above is roughly in a sequence but not strictly isolated. For example, Phase A, B, C can overlap and process in coordination.  

## The Design of TRANSFORM 

In the concept model of New KSK rollover, the TRANSFORM process is the key of the success. It require new DNS extension and functions in both authoritative and recursive side servers. Again this document will not go deep for detailed specification for TRANSFORM. More specific proposals are necessary based on the architecture of this document if the community feel it deserve further protocol development.  

### Function description

To achieve the switch between the two root system, the design of TRANSFORM should consider 2 phases : one is for "bootstrap", one is for continuous transforming.

*Bootstrap TRANSFORM is for the Initial group of health resolvers to find the new root system before the KSK-2017. Suppose that there already exist a group of resolvers who are willing and alert to switch to the new root system before the KSK-2017 rollover. They need to know the information of the new set of root server and how to switch. 

* Continuous TRANSFORM is for curable resolver who get ready with latest DNS updates months or years later (or their operator is got informed with latest information). They will be ready for TRANSFORM as well. Different from Bootstrap TRANSFORM, these resolvers will switch to a system that is already rolling to KSK-2017.

Usually resolver bootstrap itself with a "safety belt" information (SBELT). More specifically recursive resolver bootstrap using [priming query][10] with local configuration. It is OK with out-of-band of FRANSFORM by manually configuring the resolver with a new SBELT (Hint file for BIND9) containing the information of root servers in RFC5011 Ready Root System and reboot it. Given that all resolvers are already online and connected to KSK-2010 Root System, a root-rollover-like mechanism is desirable with automated and security consideration. 

### An TBD DNSSEC extension

For convenience of expression, suppose there is a to be defined DNSSEC extension, called ksk-roll-trans option, to perform both bootstrap and continuous TRANSFORM process in this context.

Suppose a resolver support this FRANSFORM DNSSEC extension, it sets the ksk-roll-trans option in the OPT RR when sending a DNSKEY query. Given that RFC8145 already enable the resolve to signaling the trust anchor information to root, the server in KSK-2010 Root System can recognized that the resolver fulfill the requirement for TRANSFORM : has both KSK-2010 and KSK-2017 as its trust anchor, and support FRANSFORM DNSSEC extension as well. 

In this case the server will deliver the information of new set of Root servers (NS RRset and glue) of RFC5011 Ready Root Server to that resolver via ksk-roll-trans option. To guarantee the integrity of data，signature of that information is required (RRSIG for NS RRset).

After validated the information of new set of root server, resolver can perform the Bootstrap TRANSFORM process. There are two options to achieve that: one is to take place the existing apex NS records in cache with the names of new root servers. Another choice is to trigger a priming query which start with the new root servers' addresses.

Continuous TRANSFORM is more complex because the TA of RFC5011 Ready Root Server is already rolled without KSK2010. So the Continuous TRANSFORM should consider a RFC5011-like KSK-rollover in design which helps the resolver to roll the key.Continuous TRANSFORM More design choices can be borrowed from RFC5011. 

## Conclusion, Future work and other Consideration

This document review and analyze the current ICANN's Root Zone KSK rollover situation and problem. Due to unclear RFC5011-rollover adoption proportion, the key issue of KSK rollover process is converted to a new technology adoption and transition problem from old-key DNSSEC to new-key DNSSEC.

Based on the analysis, the problem of KSK rollover has been decoupled into two independent goals: 1) How to keep obsolete resolvers alive after KSK rollover if there is one rollover; 2) How to roll the key for most ready resolvers to reduce the risk of old key. An new architecture for KSK rollover is proposed in this document providing a KSK-2017 transition root infrastructure with an additional paralleled root system for KSK-2017 rollover.It provides a "buffer zone" for obsolete resolver to stay and survive in the KSK-2010 DNSSEC Root system. Meanwhile Healthy Resolver and cooperated operators can roll the key independent on the unready resolver. In addition it provide a way for KSK rollover operator(ICANN) to accurately measure the transition process which will helps a decision with clear and objective metrics.   

However there are some other concerns and consideration on this plan:

* Uncertainty of time: The transition period from KSK-2010 to KSK-2017 may be very long. In Phase E a satisfied boundary (for example 80%) is not easy to reach which is dependent on the adoption of new TRANSFORM DNSSEC extension. It is also not clear that the boundary of Phase F can be satisfied.
* Create a paralleled Root system will add burden of current root operators. Now there are 700+ root instances. Then we need another 700+ instance to achieve the goal. It can be optimized with some techniques like multiple view for different resolvers to fully use the existing root infrastructure.
* Time and resource to proceed. It is a new architecture to roll the key which means in oder to achieve Phase A,B,C we need restart the work with a new implement plan, a new time line, waiting for new DNS specification etc. It requires another two or three years and necessary resources. It is not clear ICANN can afford that plan given the recent report of ICANN financial issues.

Note that before the draft document is finished there is a idea come up with my mind. Theoretically there is another possibility to switched the Dying Resolver and Curable Resolver to this new root system by renumbering addresses of the root. ICANN's KSK rollover plan can continue perform on the current root system. This approach is in question because the renumbering will affect the Healthy Resolver as well. 

## Acknowledgement 

[Alternative Rootism][11] is not a new idea. It's proposed 13 years ago by Dr. Paul Vixie suggesting IANA create an advanced services root zone. It is introduce huge controversy at that time. [Yeti testbed][12] is a prototype of this idea and running for two years.

[1]:https://www.icann.org/resources/pages/ksk-rollover
[2]: https://www.icann.org/en/system/files/files/ksk-rollover-operational-implementation-plan-22jul16-en.pdf
[3]: https://www.icann.org/news/announcement-2017-09-27-en
[4]: https://tools.ietf.org/html/rfc5011
[5]: https://www.iana.org/reports/2016/root-ksk-rollover-design-20160307.pdf 
[6]: https://tools.ietf.org/html/rfc8145
[7]: https://tools.ietf.org/html/draft-ietf-dnsop-kskroll-sentinel-00
[8]: https://www.icann.org/en/system/files/files/sac-063-en.pdf
[9]: https://www.icann.org/en/system/files/files/root-ksk-roll-postponed-17oct17-en.pdf
[10]: https://tools.ietf.org/html/rfc8109
[11]: https://yeti-dns.org/resource/workshop/paul-2015-10-Yokohama.pdf
[12]: http://yeti-dns.org/ 
[13]: https://www.icann.org/en/system/files/files/root-ksk-roll-postponed-17oct17-en.pdf


