---
title: "Unix"
authors: ["gio"]
---

#### Find Files from Shell

```
find <path> -name <filename>
```

#### Clear Memory Cache & Buffer

```
# Clear PageCache only (OK in production)
sudo sh -c "sync; echo 1 > /proc/sys/vm/drop_caches"

# Clear dentries and inodes
sudo sh -c "sync; echo 2 > /proc/sys/vm/drop_caches"

# Clear PageCache, dentries and inodes
sudo sh -c "sync; echo 3 > /proc/sys/vm/drop_caches"
```

References:

1. https://www.tecmint.com/clear-ram-memory-cache-buffer-and-swap-space-on-linux/
2. https://unix.stackexchange.com/questions/58553/how-to-clear-memory-cache-in-linux
3. https://stackoverflow.com/questions/29870068/what-are-pagecache-dentries-inodes

#### See process pid information

```
cat /proc/<pid>/status
```

#### Check system message

```
sudo tail -f /var/log/syslog
sudo dmesg
```

#### Double dash

Double dash in linux, for example:

`grep -- -v file`

Is used to signify the end of optional parameters. From that point onward positional parameters will be accepted.

See [here](https://unix.stackexchange.com/questions/11376/what-does-double-dash-mean-also-known-as-bare-double-dash)

#### awk

Split a line into multiple values. Use space as separator.

```
# Will print first column
ls -l | awk '{print $1}'

# Will print all columns
ls -l | awk '{print $0}'
```

#### xargs

Cat each files in this directory.

```
ls -l | awk '{print $9}' | xargs -I{} cat {}
```

#### cut

Considering that we have file `test.txt` containing this:

```
person-1;soccer;80
person-2;badminton;60
person-3;chess;58
```

And we want to get the list of the second column only, we can do this using `cut`:

```
cat test.txt | cut -d';' -f2
```

#### wc

Word counting in Unix.

```
# Count number of lines from stdout
sudo ls | wc -l
```

#### Communication via Socket

1. https://en.wikipedia.org/wiki/Unix_file_types
2. https://askubuntu.com/questions/372725/what-are-socket-files
3. https://unix.stackexchange.com/questions/243265/how-to-get-more-info-about-socket-file
4. https://troydhanson.github.io/network/Unix_domain_sockets.html

#### Removing an ipaddress from known_hosts

```
ssh-keygen -R <ipaddress>
```

#### Check how many file descriptors are being used

```
# Find out pid of the process first
ps aux | grep <process-name>

# Check file descriptors being used by a particular process (Opt 1)
lsof -a -p <pid>

# Check file descriptors being used by a particular process (Opt 2)
cd /proc/28290/fd

# Then do
ls -l | less

# Or
ls -l | wc -l

# Check file descriptors being used (total)
lsof | wc -l
```

See also [here](https://www.cyberciti.biz/tips/linux-procfs-file-descriptors.html)

#### Measuring request and response time using curl

```
curl -X <request-type> \
  -w %{time_connect}:%{time_starttransfer}:%{time_total} \
  server:port \
  -d <payload>
```

See also [here](https://stackoverflow.com/questions/18215389/how-do-i-measure-request-and-response-times-at-once-using-curl)
