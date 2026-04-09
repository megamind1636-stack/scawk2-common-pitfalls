# AWK Notes — SE2001 TA Session (Swati Ma'am)

---

## 1. AWK Program Structure

AWK ka program 3 blocks mein hota hai:

```awk
BEGIN {
    # Initialization only
    # File yahan read NAHI hoti
    # Variables initialize karo yahan
}

{
    # Body / Main block
    # File line by line yahan read hoti hai
    # Filtering, computation, manipulation sab yahan
}

END {
    # Sab lines read ho jaane ke baad chalta hai
    # Final totals, averages, results yahan print karo
}
```

**Important rules:**
- Koi bhi block skip kar sakte ho
- Multiple BEGIN/END blocks likh sakte ho — valid hai, but bad practice
- Body block ek implicit loop hai — file ki har line ke liye ek baar chalta hai

---

## 2. Field Variables (Columns)

```
File: Alice  32  Engineer  60000
      $1     $2  $3        $4
```

| Variable | Matlab |
|----------|--------|
| `$1` | pehla column |
| `$2` | doosra column |
| `$3` | teesra column |
| `$0` | poori line (entire record) |
| `NF` | Number of Fields — us line mein kitne columns hain |
| `NR` | Number of Records — abhi tak kitni lines read hui |

> **Note:** `$1`, `$2`... column numbers hain, pass kiye hue variables nahi.

---

## 3. AWK vs Bash — Syntax Comparison

| Feature | Bash `.sh` | AWK |
|---------|-----------|-----|
| Print | `echo "text"` | `print "text"` |
| Print no newline | `echo -n "text"` | `printf "text"` |
| if block | `if [ cond ]; then ... fi` | `if (cond) { ... }` |
| for loop | `for i in ...; do ... done` | `for (i=1; i<=n; i++) { ... }` |
| while loop | `while [ cond ]; do ... done` | `while (cond) { ... }` |
| Variable assign | `x=5` | `x = 5` (spaces allowed!) |
| String compare | `[ "$a" == "b" ]` | `$3 == "Engineer"` |

**AWK = C-style syntax.** Koi `then`, `fi`, `do`, `done` nahi. Bas parentheses aur curly braces.

---

## 4. print vs printf

```awk
# print — automatically newline add karta hai
{ print "Hello World" }
{ print "Name:", $1 }        # comma = space separator in output
{ print $1, $2 }             # "Alice 32"
{ print $1 $2 }              # "Alice32" — no space (concatenation)
{ print $1, "-", $2 }        # "Alice - 32"

# printf — bash jaisa, no automatic newline
{ printf "Name: %s, Age: %d\n", $1, $2 }
{ printf "%.2f\n", avg }     # 2 decimal places
```

> **Pitfall:** Variable ko quotes ke andar mat likho!
> ```awk
> END { print "Total sum" }       # WRONG — prints literal "Total sum"
> END { print "Total sum:", sum } # CORRECT — label + actual value
> ```

---

## 5. NR — Number of Records

```awk
# CORRECT — END block mein use karo total ke liye
END { print "Total lines:", NR }

# WRONG — body mein NR print karo toh running counter milega
{ print NR }    # 1, 2, 3, 4... har line pe alag value
```

**Key points:**
- NR ek running counter hai — jaise hi ek line read hoti hai, NR ++ ho jaata hai
- Condition false ho toh bhi NR badhta rehta hai
- Total line count chahiye? Sirf `END` block mein `NR` use karo

---

## 6. Common Operations

### Print specific columns
```awk
{ print $1, $3 }           # name aur position
```

### Filter by condition
```awk
$2 > 25 { print $1, $2, $3 }    # age > 25 wale
```

### Convert to uppercase / lowercase
```awk
{ print toupper($0) }      # poori line uppercase
{ print tolower($1) }      # sirf naam lowercase
```

### Exact string match
```awk
$3 == "Engineer" { print $1, $4 }    # case sensitive, exact spelling
```

### Sum of a column
```awk
{ sum += $4 }              # sum = sum + $4 (shortcut)
END { print "Total:", sum }
```

### Average
```awk
NR > 1 { total += $4; count++ }
END { print "Average:", total / count }
```

### Maximum value
```awk
BEGIN { max = 0 }
NR > 1 {
    if ($2 > max) max = $2
}
END { print "Max age:", max }
```

### Print all engineers with salary
```awk
$3 == "Engineer" { print $1, $4 }
```

---

## 7. Regex Matching

```awk
# ~ means "matches pattern"
$3 ~ /Engineer/    { print }      # $3 mein "Engineer" hai
$3 ~ /^Eng/        { print }      # $3 "Eng" se start hota hai
$0 ~ /Alice/       { print }      # poori line mein "Alice" hai

# !~ means "does NOT match"
$0 !~ /^#/         { print }      # line # se start nahi hoti

# == means EXACT match only (no regex)
$3 == "Engineer"   { print }      # exact "Engineer" hona chahiye
```

> **Pitfall:** `$3 == /pattern/` — ye allowed nahi hai. `==` ke saath regex mat use karo.
> Regex ke liye hamesha `~` use karo.

---

## 8. Skipping the Header Line

**Method 1 — NR > 1 (sabse simple)**
```awk
NR > 1 { print $1, $2 }
```

**Method 2 — next keyword**
```awk
NR == 1 { next }         # line 1 skip karo
{ print $1, $2 }         # baki sab ke liye chalta hai
```

**Method 3 — Regex pattern**
```awk
$0 !~ /^#/ { print toupper($0) }    # # se shuru hone wali lines skip
```

> **Why skip karna zaroori hai?**
> Agar header skip nahi kiya aur max/min dhundh rahe ho, toh AWK string "Employee" ko number se compare karega. String ko higher value milti hai — toh "Employee" hi max ban jaata hai. No error, sirf wrong output.

---

## 9. if / else if / else

```awk
NR > 1 {
    if ($2 > 30) {
        print $1, "senior"
    } else if ($2 > 25) {
        print $1, "mid"
    } else {
        print $1, "junior"
    }
}
```

Bash jaisa hi hai, bas `then` aur `fi` nahi — C-style parentheses aur curly braces.

---

## 10. Loops

### for loop
```awk
# Basic
BEGIN {
    for (i = 1; i <= 5; i++) {
        print i
    }
}

# Har line ke saare fields print karo
{
    for (i = 1; i <= NF; i++) {
        print $i      # $i — variable ke andar value se field access
    }
}
```

### while loop
```awk
BEGIN {
    i = 1
    while (i <= 5) {
        print i
        i++
    }
}
```

### do-while loop (bash mein nahi hota!)
```awk
BEGIN {
    i = 1
    do {
        print i
        i++
    } while (i <= 5)
}
# Difference: body pehle chalti hai, condition baad mein check hoti hai
# Matlab: kam se kam ek baar toh chalega hi
```

---

## 11. break, continue, next — Differences

Ye teeno alag hain — confuse mat karo:

| Command | Kahan kaam karta hai | Kya karta hai |
|---------|----------------------|---------------|
| `break` | `for`/`while` loop ke andar | Poora loop band kar deta hai |
| `continue` | `for`/`while` loop ke andar | Sirf current iteration skip, loop chalta rehta hai |
| `next` | Body level (loop ke bahar) | Current line skip, agli line pe ja |

### break — loop se bilkul bahar
```awk
{
    n = $1
    count = 0
    for (i = 2; i < n; i++) {
        if (n % i == 0) {
            count++
            break        # divisor mil gaya, aage check karne ki zaroorat nahi
        }
    }
    if (count == 0) print n, "is prime"
    else            print n, "is not prime"
}
```

### continue — sirf is iteration skip karo
```awk
{
    for (i = 1; i <= 10; i++) {
        if (i % 2 == 0) {
            continue     # even numbers skip — loop band nahi hoga
        }
        print i          # sirf odd: 1, 3, 5, 7, 9
    }
}
```

### next — current line skip, agli line pe ja
```awk
NR == 1        { next }     # header skip
$2 < 25        { next }     # age < 25 wale skip
$4 < 30000     { next }     # salary < 30000 wale skip
{ print $1, $2, $4 }        # jo bacha wo print

# next = bash ke continue jaisa hai
# but loops ke liye nahi — AWK ke implicit LINE loop ke liye hai
```

> **next ka fayda:** Jab zyada conditions hon, pehle saari rejections likho `next` se, end mein ek clean action. Readable rehta hai.

---

## 12. Multiple Conditions

```awk
# AND — && — dono conditions true honi chahiye
NR > 1 && $2 > 25 && $4 > 30000 { print $1 }

# OR — || — koi ek condition true ho
NR > 1 && ($2 > 25 || $4 > 50000) { print $1 }

# OR conditions ko parentheses mein wrap karo — readability ke liye
```

---

## 13. Initialization — Kahan Karna Hai?

Ye sabse common confusion hai. Rule simple hai:

| Kya initialize karna hai | Kahan karo |
|--------------------------|------------|
| Running totals (sum, max, min) | `BEGIN` block |
| Cross-line counters | `BEGIN` block |
| Per-line fresh counters (prime check etc.) | Body ke andar |

```awk
# WRONG — max har line pe 0 ho jaata hai
{
    max = 0              # body mein — BAD
    if ($2 > max) max = $2
}

# CORRECT
BEGIN { max = 0 }        # ek baar initialize
NR > 1 {
    if ($2 > max) max = $2
}

# Per-line counter — body mein initialize karna SAHI hai
{
    count = 0            # har number ke liye fresh start chahiye
    for (i = 2; i < $1; i++)
        if ($1 % i == 0) { count++; break }
    if (count == 0) print $1, "prime"
}
```

---

## 14. Running AWK

```bash
# Ek-liner — terminal mein seedha
awk '{ print $1, $3 }' file.txt

# Script file se
gawk -f script.awk file.txt

# Script file mein shebang
#!/usr/bin/gawk -f

# Execute permission dena zaroori hai script ke liye
chmod +x script.awk
./script.awk file.txt
```

---

## 15. cut Command (Quick Reference)

```bash
cut -c3           # 3rd character
cut -c2,7         # 2nd aur 7th character
cut -c2-7         # 2nd se 7th tak (range)
cut -c-4          # pehle 4 characters (1 to 4)
cut -c30-         # 30th se end tak

cut -f1-3         # TSV file: fields 1 to 3 (tab auto-detect)
cut -d' ' -f4     # space delimiter, 4th field
cut -d',' -f1-3   # comma delimiter (CSV), first 3 fields
cut -f2-          # 2nd field se end tak
```

---

## 16. sort Command (Quick Reference)

```bash
sort file.txt              # alphabetical (lexicographic) — default
sort -r file.txt           # reverse order
sort -n file.txt           # numeric sort — NUMBERS KE LIYE ZAROORI
sort -nr file.txt          # numeric + descending

# Column-wise sort
sort -k2 -n -r file.txt              # 2nd column, numeric, descending

# Tab-separated file
sort -k2 -n -r -t$'\t' file.txt
#              ^ dollar sign zaroori — \t do characters hain, dollar unhe real tab banata hai

# Pipe-separated file
sort -k2 -n -r -t'|' file.txt
# Single character separator = no dollar needed
```

> **Yaad rakh:** Default sort = dictionary order. 100 comes before 9 kyunki '1' < '9'. Numbers sort karne ke liye hamesha `-n` lagao.

---

## 17. Common Pitfalls Summary

| # | Pitfall | Fix |
|---|---------|-----|
| 1 | Variable initialize kiya body mein (har line pe reset) | `BEGIN` mein initialize karo |
| 2 | `NR` body mein print kiya total ke liye | `END` mein print karo |
| 3 | Header wali file mein `NR` as denominator | Apna `count` variable rakho |
| 4 | Header skip nahi kiya → string "max" ban jaati hai | `NR > 1` ya `next` use karo |
| 5 | Variable quotes ke andar likh diya | Variable bahar rakho: `print "Label:", var` |
| 6 | `==` ke saath regex use kiya | Regex ke liye `~` use karo |
| 7 | Per-line counter `BEGIN` mein daal diya | Body mein initialize karo (har line ke liye fresh start) |
| 8 | Numbers sort karte waqt `-n` bhool gaye | Hamesha `-n` lagao numeric data ke liye |
| 9 | Tab separator mein dollar sign bhool gaye | `-t$'\t'` — dollar zaroori hai |

---

*Session: SE2001 TA Session — Swati Jain (21 Nov 2025)*
