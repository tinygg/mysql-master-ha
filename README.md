# mysql-master-ha

A primary objective of MHA is automating master failover and slave promotion within short (usually 10-30 seconds) downtime, without suffering from replication consistency problems, without spending money for lots of new servers, without performance penalty, without complexity (easy-to-install), and without changing existing deployments.

MHA also provides a way for scheduled online master switch: changing currently running master to a new master safely, within a few seconds (0.5-2 seconds) of downtime (blocking writes only).

MHA provides the following functionality, and can be useful in many deployments where requirements such as high availability, data integrity, almost non-stop master maintenance are desired.

- Automated master monitoring and failover

MHA has a functionality to monitor MySQL master in an existing replication environment, detecting master failure, and doing master failover automatically. Even though some of slaves have not received the latest relay log events, MHA automatically identifies differential relay log events from the latest slave, and applies differential events to other slaves. So all slaves can be consistent. MHA normally can do failover in seconds (9-12 seconds to detect master failure, optionally 7-10 seconds to power off the master machine to avoid split brain, a few seconds for applying differential relay logs to the new master, so total downtime is normally 10-30 seconds). In addition, you can define a specific slave as a candidate master (setting priorities) in a configuration file. Since MHA fixes consistencies between slaves, you can promote any slave to a new master and consistency problems (which might cause sudden replication failure) will not happen.

- Interactive (manual) Master Failover

You can also use MHA for just failover, not for monitoring master. You can use MHA for master failover interactively.

- Non-interactive master failover

Non-interactive master failover (not monitoring master, but doing failover automatically) is also supported. This feature is useful especially when you have already used a software that monitors MySQL master. For example, you can use Pacemaker(Heartbeat) for detecting master failure and virtual ip address takeover, and use MHA for master failover and slave promotion.

- Online switching master to a different host

In many cases, it is necessary to migrate an existing master to a different machine (i.e. the current master has H/W problems on RAID controller or RAM, you want to replace with faster machine, etc). This is not a master crash, but scheduled master maintenance is needed to do that. Scheduled master maintenance causes downtime (at least you can not write master) so should be done as quickly as possible. On the other hand, you should block/kill current running sessions very carefully because consistency problems between different masters might happen (i.e "updating master1, updating master 2, committing master1, getting error on committing master 2" will result in data inconsistency). Both fast master switch and graceful blocking writes are required. MHA provides a way to do that. You can switch master gracefully within 0.5-2 seconds of writer block. In many cases 0.5-2 seconds of writer downtime is acceptable and you can switch master even without allocating scheduled maintenance window. This means you can take actions such as upgrading to higher versions, faster machine, etc much more easily.
