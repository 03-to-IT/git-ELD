
```
$ echo "There were $(grep -c ' sudo: ' /var/log/auth.log) attempts to use sudo, $(grep -c ' sudo: .*authentication failure' /var/log/auth.log) of which failed."
There were 17 attempts to use sudo, 1 of which failed.
```
### Print lines within time range
```
awk '/01:05:/,/01:20:/' access.log
```

---
### Print within time range, when matching pattern "POST"
```
awk '/01:05:/,/01:20:/ { if (/POST/) print }' access.log
```

---
More example options: 

    [^[:space:]]+ matches one or more non-space characters.
    
    \[[^\]]+\]matches one or more characters (including spaces) enclosed in square brackets.
    
    "[^"]+" matches one or more characters (including spaces) enclosed in double quotes.
    
    ([^[:space:]]+|\[[^\]]+\]|"[^"]+") ORs those together and matches any one of the above, which covers all of the field formats in my access log.
    
    [[:space:]]+ matches one or more space.
    
    (([^[:space:]]+|\[[^\]]+\]|"[^"]+")[[:space:]]+) concatenates those, so that we end up with a pattern that matches a field and then a space.
    
    Adding {7} to the end (and the --re-interval flag) matches the seventh occurrence of a field followed by a space on each line. You'll probably need to adjust this to print the correct field for your log format.
    
    match matches the first argument ($0 — the current line) against the regular expression in the second argument, and stores the captured matches in an array named by the third argument (m).
    
    m[2] prints the second captured group, counting left parentheses. In this case, that means the matched field, without the trailing spaces.
    
    sort -nk 1 sorts numerically on the first field of the input, in ascending order, so the biggest requests are at the end.

---
You can do pretty much anything with apache log files with awk alone. Apache log files are basically whitespace separated, and you can pretend the quotes don't exist, and access whatever information you are interested in by column number. The only time this breaks down is if you have the combined log format and are interested in user agents, at which point you have to use quotes (") as the separator and run a separate awk command. The following will show you the IPs of every user who requests the index page sorted by the number of hits:
```
awk -F'[ "]+' '$7 == "/" { ipcount[$1]++ }
    END { for (i in ipcount) {
        printf "%15s - %d\n", i, ipcount[i] } }' logfile.log
```
$7 is the requested url. You can add whatever conditions you want at the beginning. Replace the '$7 == "/" with whatever information you want.

If you replace the $1 in (ipcount[$1]++), then you can group the results by other criteria. Using $7 would show what pages were accessed and how often. Of course then you would want to change the condition at the beginning. The following would show what pages were accessed by a user from a specific IP:
```
awk -F'[ "]+' '$1 == "1.2.3.4" { pagecount[$7]++ }
    END { for (i in pagecount) {
        printf "%15s - %d\n", i, pagecount[i] } }' logfile.log
```
You can also pipe the output through sort to get the results in order, either as part of the shell command, or also in the awk script itself:
```
awk -F'[ "]+' '$7 == "/" { ipcount[$1]++ }
    END { for (i in ipcount) {
        printf "%15s - %d\n", i, ipcount[i] | sort } }' logfile.log
```
The latter would be useful if you decided to expand the awk script to print out other information. It's all a matter of what you want to find out. These should serve as a starting point for whatever you are interested in.

---
IP counts for access.log
```
cat log | grep -o "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}" | sort -n | uniq -c | sort -n
```

---
netstat - active connections
```
netstat -an | awk '{print $5}' | grep -o "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}" | egrep -v "(`for i in \`ip addr | grep inet |grep eth0 | cut -d/ -f1 | awk '{print $2}'\`;do echo -n "$i|"| sed 's/\./\\\./g;';done`127\.|0\.0\.0)" | sort -n | uniq -c | sort -n
```

---

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEyNjEyMDA2MV19
-->