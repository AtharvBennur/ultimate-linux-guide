# Text Processing

## What Is Text Processing?

Text processing searches, transforms, extracts, sorts, and summarizes text files. DevOps engineers use it for logs, configuration files, CSV data, command output, and automation.

Commands can be combined with pipes and redirection:

- `|` sends standard output to another command.
- `>` replaces a file with command output.
- `>>` appends command output to a file.
- `<` reads input from a file.

## grep

```bash
grep "ERROR" app.log
grep -i "error" app.log
grep -r "8080" /etc/nginx/
grep -v "INFO" app.log
grep -n "fatal" app.log
```

| Option | Meaning |
|---|---|
| `-i` | Ignore case |
| `-r` | Search recursively |
| `-v` | Show non-matching lines |
| `-n` | Show line numbers |

Extended regular expressions:

```bash
grep -E "ERROR|FATAL" app.log
grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}" app.log
```

## sed

`sed` stream-edits text:

```bash
sed 's/old/new/' file.txt
sed 's/old/new/g' file.txt
sed -n '10,20p' file.txt
sed -i 's/Listen 80/Listen 8080/' /tmp/nginx.conf
```

The `g` flag replaces every match on a line. Use `-i.bak` when creating a backup before an in-place edit:

```bash
sed -i.bak 's/old/new/g' file.txt
```

## cut

`cut` extracts columns with separators or character positions:

```bash
cut -d ':' -f 1,3 /etc/passwd
cut -d ',' -f 2 data.csv
cut -c 1-10 file.txt
```

## awk

`awk` processes records into fields. By default, whitespace separates fields. `$1` is the first field, `$2` is the second, and `$NF` is the final field.

```bash
awk '{print $1, $3}' file.txt
awk -F ':' '{print $1, $6}' /etc/passwd
awk '/ERROR/ {print $0}' app.log
awk '{count[$1]++} END {for (ip in count) print ip, count[ip]}' access.log
```

Practical column example:

```bash
df -h | awk 'NR>1 {print $1, $5}'
```

## sort, uniq, and wc

```bash
sort file.txt
sort -r file.txt
sort -k 2 -n file.txt
uniq file.txt
sort file.txt | uniq -c
sort file.txt | uniq -d
wc -l file.txt
wc -w file.txt
wc -c file.txt
```

`uniq` only removes adjacent duplicates, so sort the input first.

## tr

```bash
tr 'a-z' 'A-Z' < file.txt
tr -d '\r' < file.txt
tr ',' ' ' < data.csv
```

## xargs

`xargs` converts input into command arguments:

```bash
find /tmp -type f -name "*.log" | xargs rm
printf "a\nb\nc\n" | xargs -n 1 echo
find . -type f -print0 | xargs -0 grep "TODO"
```

Use `-0` with `find -print0` to support file names containing spaces.

## find

```bash
find /var/log -type f -name "*.log"
find /home -type f -size +100M
find /tmp -type f -mtime +7
find /srv/app -type f -perm 0777
```

## Practical DevOps Examples

### Extract ERROR Messages

```bash
grep -nEi "error|fatal|exception" /var/log/myapp/app.log
```

### Find the Top IP Addresses in Nginx Access Logs

Assuming the IP is the first field:

```bash
awk '{print $1}' /var/log/nginx/access.log |
sort |
uniq -c |
sort -nr |
head
```

### Extract a Configuration Value

```bash
grep -Rni "Listen" /etc/nginx/
```

### Find Files Larger Than a Given Size

```bash
find /var -type f -size +500M -exec ls -lh {} \;
```

### Count HTTP Status Codes

```bash
awk '{print $9}' /var/log/nginx/access.log |
sort |
uniq -c |
sort -nr
```

## Combining Commands

Example pipeline to find warning lines for a service and save a report:

```bash
journalctl -u nginx --since today |
grep -iE "warning|error" |
tee /tmp/nginx-warnings.log |
wc -l
```

## Common Mistakes

- Using `uniq` without sorting.
- Using a broad regular expression and hiding useful context.
- Running `sed -i` on configuration without a backup.
- Using `rm` in `xargs` without testing the input list.
- Processing binary files with text utilities.

## Interview Questions

**Q: What is a pipe?**  
It sends the output of one command to the input of another command.

**Q: What does `awk '{print $1}'` do?**  
It prints the first whitespace-separated field from every input line.

**Q: What is the difference between `>` and `>>`?**  
`>` overwrites a file; `>>` appends to it.

## Exercises

1. Extract `ERROR` and `FATAL` lines from a log.
2. Find the 10 most common IP addresses.
3. Extract usernames and home directories from `/etc/passwd`.
4. Find files larger than 100 MB under `/var`.
5. Replace `old.example.com` with `new.example.com` in a copied configuration file.

**Related:** [Log Management](../15-log-management/) | [Monitoring](../08-monitoring/)
