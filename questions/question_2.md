# Question 2

An "inordinately complex command" 😲
```
for a in `yes | nl | head -50 | cut -f 1`; do \
    head -$(($a*2)) inputfile | tail -1 | \
    awk 'BEGIN{FS="\t"}{print $2}' | xargs wget -c 2> /dev/null;
done
```

## What can you tell about the expected contents of the input file?

I would expect the format of the input file to resemble something similar to this:
```

playstation	https://www.playstation.com/

google	https://www.google.com/

facebook	https://www.facebook.com/
```
Key points:
- All odd rows (1,3,5,...) aren't utilized, I left them blank.
- 100 total rows (possibly unique).
- Each row has at least two fields delimited by tab characters.
- The first field is inconsequential.
- The second field should resolve as a web address.
- Fields >= 3 are inconsequential.

## What does the command do?

Breaking this command down and explaining each line:

```
yes | nl | head -50 | cut -f 1
```
This line sends a list of integers from 1 to 50 to stdout.
- `yes` will write "y" to stdout indefinitely.
- `nl` will read from stdin, prepend the file line and write to stdout.
- `head -50` will halt the `yes` command at 50 iterations. It also renumbers the lines 1-50 instead of the original stdout line numbers.
- `cut -f 1` will remove the first index of the list of words in stdout. In this example "y". Then write to stdout.

----

```
for a in `yes | nl | head -50 | cut -f 1`; do
    ...
done
```
A for loop iterating from 1 to 50.

---

```
head -$(($a*2)) inputfile | tail -1
```
This line will read from the file (`inputfile`) and retrieve only the a*2. e.g. `a=3` will return line 6.

- `head -$(($a*2))` accesses variable `a` from the for loop, multiplies it by 2 then reads all lines from 0 to the resultant number of lines starting at the top of the file.

- `| tail -1` returns only the last line from the stdin.

---
```
awk 'BEGIN{FS="\t"}{print $2}' | xargs wget -c 2> /dev/null;
```
This line reads from stdin, extracts the second field of the input line and performs a HTTP GET request using this value. Any errors are written to `/dev/null` effectively discarding them.

- `awk 'BEGIN{FS="\t"}{print $2}'` extracts the second field of the line, delimited by tabs from stdin and writes it to stdout.

- `xargs wget -c` reads items from stdin and executes `wget` to HTTP GET the provided resource URL from the internet. `-c` will resume a previously interrupted download.

- `2> /dev/null` discards any errors.

## How would you simplify it?

I try to avoid shell scripts where possible and would likely write something like this in Python with unit tests!

For the purpose of this challenge, I would simplify it like this:

```
head -100 inputfile | awk 'BEGIN{FS="\t"} NR % 2 == 0 {print $2}' | xargs wget -c 2> /dev/null
```
- `head -100 inputfile` reads the top 100 lines from `inputfile`
- `awk 'NR % 2 == 0 {print $2}'` Utilise awk to extract the second field of every even-numbered line. The `FS` option isn't required but I retained it to document the `inputfile` delimiters.

- `xargs wget -c` reads from stdin, parses the input and provides it as a parameter to `wget` as a URL. `wget` then downloads the resource.
