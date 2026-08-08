# Linux Disk & Memory Commands — Real DevOps Scenario

Imagine a production server suddenly becomes slow.

You SSH into the server and check three things:

---

## 1. `df` — Is the Disk Full?

Check available disk space:

```bash
df -h

Example:

Filesystem      Size  Used Avail Use%
/dev/sda1        50G   47G    3G  94%

The disk is 94% full.

Now find what is consuming the space.

2. du — What Is Using the Disk?

Check directory sizes:

du -sh /*

Example:

2.1G    /home
8.5G    /opt
32G     /var
1.2G    /usr

/var is using the most space.

Investigate further:

du -sh /var/*

Example:

28G     /var/log

Now you know that logs are consuming most of the disk.

Remember
df → Is the disk full?
du → What is filling the disk?
3. free — Is RAM Running Out?

Check memory:

free -h

Example:

              total   used   free
Mem:            8Gi    7.5Gi  500Mi
Swap:           2Gi    1.8Gi  200Mi

Memory usage is very high.

Find processes consuming resources:

top

or:

htop
Real Incident Workflow

Production server is slow:

ssh user@server-ip

Check disk:

df -h

If disk is nearly full:

du -sh /*

Investigate the largest directory:

du -sh /var/*

Check memory:

free -h

Find resource-heavy processes:

top
The 3 Commands to Remember
Command	Question
df -h	Is the disk full?
du -sh	What is using the disk?
free -h	Is RAM running out?
Production issue
       ↓
     df -h
       ↓
Disk full?
   ↓       ↓
  Yes      No
   ↓       ↓
 du -sh   free -h
   ↓       ↓
Find      Check
files     memory
