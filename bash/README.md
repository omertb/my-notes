# Bash Terminal Notes

#### Change filenames in batch

```
# converting files ending with running.config to ending with .conf.
$ for file in *.config; do mv "$file" "${file/running.config/.conf}"; done

```
#### Delete Files Older Than x Days
```
# Delete files older than 2 years file name beginning with "@":
$ find . -mtime +730 -type f -name "@*" -print
$ find . -mtime +730 -type f -name "@*" -delete

Print and delete files  older than 60 days and those are beginning with "dhcpd" and ending with "conf":
$ find . -mtime +60 -type f -name "dhcpd*conf" -print
$ find . -mtime +60 -type f -name "dhcpd*conf" -delete

```

