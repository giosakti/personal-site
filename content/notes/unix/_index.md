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

#### Communication via Socket

1. https://en.wikipedia.org/wiki/Unix_file_types
2. https://askubuntu.com/questions/372725/what-are-socket-files
3. https://unix.stackexchange.com/questions/243265/how-to-get-more-info-about-socket-file
4. https://troydhanson.github.io/network/Unix_domain_sockets.html
