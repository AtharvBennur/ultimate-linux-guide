# File management in Linux

### File and Directory Management
1. **`ls`** – Lists files and directories in the current location.
2. **`cd /path/to/directory`** – Changes the working directory.
3. **`pwd`** – Prints the current working directory.
4. **`mkdir new_folder`** – Creates a new directory.
5. **`rmdir empty_folder`** – Removes an empty directory.
6. **`rm file.txt`** – Deletes a file.
7. **`rm -r folder`** – Deletes a folder and its contents.
8. **`cp file1.txt file2.txt`** – Copies a file.
9. **`cp -r dir1 dir2`** – Copies a directory recursively.
10. **`mv old_name new_name`** – Moves or renames a file or directory.

### File Viewing and Editing
11. **`cat file.txt`** – Displays file content.
12. **`tac file.txt`** – Displays file content in reverse order.
13. **`less file.txt`** – Opens a file for viewing with scrolling support.
14. **`more file.txt`** – Similar to `less`, but only moves forward.
15. **`head -n 10 file.txt`** – Displays the first 10 lines of a file.
16. **`tail -n 10 file.txt`** – Displays the last 10 lines of a file.
17. **`nano file.txt`** – Opens a simple text editor.
18. **`vi file.txt`** – Opens a powerful text editor.
19. **`echo 'Hello' > file.txt`** – Writes text to a file, overwriting existing content.
20. **`echo 'Hello' >> file.txt`** – Appends text to a file without overwriting.

## Wildcards

The shell expands wildcard patterns before it runs a command:

| Pattern | Meaning |
|---|---|
| `*` | Any number of characters |
| `?` | Exactly one character |
| `[abc]` | One character from the listed set |

Examples:

```bash
ls *.txt
ls report-?.log
cp *.conf /tmp/configs/
```

Quote a pattern when you want a program such as `find` to receive it literally.

## Find Files

`find` searches a directory tree by name, type, size, age, and other criteria:

```bash
find "$HOME" -type f -name "*.log"
find /etc -type f -name "*.conf"
find "$HOME" -type f -size +100M
find /tmp -type f -mtime +7
find "$HOME" -type f -name "*.tmp" -exec ls -l {} +
```

`-type f` selects regular files, `-name` matches a filename pattern,
`-size +100M` selects files larger than 100 MiB, and `-mtime +7` selects files
last modified more than seven 24-hour periods ago. Use a directory you can
read; permissions may limit results.

## Hard and Symbolic Links

A hard link is another directory entry for the same file data. A symbolic
(soft) link stores a path to its target.

```bash
ln report.txt report-hardlink
ln -s report.txt report-shortcut
ls -li report.txt report-hardlink report-shortcut
```

Hard links normally cannot cross filesystems and are generally not made for
directories. A symbolic link can point across filesystems and can point to a
directory, but it can become dangling if its target is moved or removed. When
using a relative target for a symbolic link, the path is interpreted relative
to the link's directory.

## Inspect Files and Disk Usage

```bash
stat report.txt
file report.txt
du -sh "$HOME"
du -h --max-depth=1 "$HOME"
```

- `stat` shows file metadata such as size, permissions, and timestamps.
- `file` reports the type it detects from the file's contents.
- `du` estimates disk space used by files and directories. `-s` gives a
  summary, while `-h` uses readable units.

## Archiving and Compression

`tar` groups files into an archive; the `-z` option uses gzip compression:

```bash
tar -cf project.tar project/
tar -tf project.tar
tar -xf project.tar
tar -czf project.tar.gz project/
tar -xzf project.tar.gz
```

The options above create, list, extract, create a gzip-compressed archive, and
extract a gzip-compressed archive, respectively. Extract archives only into a
trusted location and inspect their contents first when needed.

`gzip` compresses an individual file and replaces it with a `.gz` file;
`gunzip` decompresses it:

```bash
gzip report.log
gunzip report.log.gz
```

ZIP archives can contain multiple files and directories. Install the tools if
they are not already available:

```bash
# Debian/Ubuntu
sudo apt install zip unzip

# RHEL-family
sudo dnf install zip unzip
```

Create and extract an archive:

```bash
zip -r project.zip project/
unzip project.zip
```

## Synchronize Files with `rsync`

`rsync` copies and synchronizes files locally or between systems. Install it
when needed:

```bash
# Debian/Ubuntu
sudo apt install rsync

# RHEL-family
sudo dnf install rsync
```

Copy a directory's contents into another directory:

```bash
rsync -av project/ project-copy/
```

The trailing slash on `project/` means to copy its contents into
`project-copy/`. Review source and destination carefully before adding options
that delete destination files.

## Compare Files with `diff`

`diff` compares text files and can show changes in a format commonly used by
patch tools:

```bash
diff -u original.conf updated.conf
```

No output means the files match. A nonzero status is expected when files
differ, so scripts should account for that when using `diff` as a test.

## Practical Scenario: Prepare and Compare an Application Backup

For an application stored in `/srv/app`, create a compressed archive, inspect
its file list, and compare two configuration versions:

```bash
tar -czf "$HOME/app-backup.tar.gz" -C /srv app
tar -tzf "$HOME/app-backup.tar.gz"
diff -u /srv/app/config.conf /srv/app/config.conf.new
```

To copy the application tree to a separate destination, first confirm that the
destination is correct and available:

```bash
rsync -av /srv/app/ /backup/app/
```

Use backups appropriate to the system and application; a local archive alone
does not protect against disk failure.

## Common Mistakes

- Forgetting that the shell expands an unquoted wildcard before running a command.
- Treating a symbolic link as a copy of the target file.
- Assuming a hard link can cross filesystem boundaries.
- Confusing an archive with a compressed archive; plain `tar` does not compress.
- Extracting an untrusted archive into a sensitive directory.
- Misreading a nonzero `diff` status: different files are a normal comparison result.
- Using `rsync` options without checking which path is the source and which is the destination.

## Interview Questions

**Q: What is the difference between a hard link and a symbolic link?**

A hard link is another name for the same file data; a symbolic link stores a
path to a target.

**Q: What is the difference between `tar` and `gzip`?**

`tar` packages files into an archive. `gzip` compresses a stream or file;
`tar -czf` combines archiving with gzip compression.

**Q: What does a trailing slash on an `rsync` source directory mean?**

It copies the directory's contents into the destination rather than placing
the source directory itself inside it.

## Exercises

1. List all `.log` files under your home directory with `find`.
2. Create a hard link and a symbolic link to a test file, then inspect them
   with `ls -li`.
3. Use `stat`, `file`, and `du` to inspect a file and its containing directory.
4. Create a tar archive and a gzip-compressed tar archive of a test directory,
   then list their contents.
5. Copy a directory with `rsync` and compare two text files with `diff -u`.

