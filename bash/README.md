# Bash Terminal Notes

#### Change filenames in batch

```Shell
# converting files ending with running.config to ending with .conf.
$ for file in *.config; do mv "$file" "${file/running.config/.conf}"; done

```
#### Delete Files Older Than x Days
```Shell
# Delete files older than 2 years file name beginning with "@":
$ find . -mtime +730 -type f -name "@*" -print
$ find . -mtime +730 -type f -name "@*" -delete

Print and delete files  older than 60 days and those are beginning with "dhcpd" and ending with "conf":
$ find . -mtime +60 -type f -name "dhcpd*conf" -print
$ find . -mtime +60 -type f -name "dhcpd*conf" -delete

```

#### Monitor Disk Status
* Validated for CentOS:
```Shell
#!/bin/bash
CURRENT=$(df / | grep / | awk '{ print $4}' | sed 's/%//g')
THRESHOLD=90
if [ "$CURRENT" -gt "$THRESHOLD" ] ; then
    /bin/mail -r "diskmon" -s 'Disk Space Alert' somemail@example.com << EOF
Your root partition remaining free space is critically low. Used: $CURRENT%
EOF
fi
```
*Validated for Ubuntu
```Shell
#!/bin/bash
CURRENT=$(df / | grep / | awk '{ print $5}' | sed 's/%//g')
THRESHOLD=90
if [ "$CURRENT" -gt "$THRESHOLD" ] ; then
    /usr/bin/mail -a "From: diskmon" -s 'Disk Space Alert' somemail@example.com << EOF
Your root partition remaining free space is critically low. Used: $CURRENT%
EOF
fi
```

