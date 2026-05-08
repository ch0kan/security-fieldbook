# Bash Scripting

Bash scripting is used to automate Linux command-line tasks. A script can take user input, store values in variables, make decisions, repeat actions with loops, and save output for later analysis.

For cybersecurity work, Bash is useful for tasks such as host discovery, log parsing, file checks, simple reconnaissance, backups, and tool automation.

---

## Script Structure

A Bash script usually begins with a **shebang**, which tells Linux what interpreter should run the file.

```bash
#!/bin/bash
```

Common interpreter examples:

| Shebang | Meaning |
|---|---|
| `#!/bin/bash` | Run the script with Bash |
| `#!/usr/bin/env python3` | Run the script with Python 3 |

After creating a script, make it executable:

```bash
chmod +x script.sh
```

Then run it:

```bash
./script.sh
```

---

## Arguments and Special Variables

Arguments are values passed to a script from the command line.

```bash
./scan.sh example.com
```

In this example, `example.com` becomes `$1`.

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1`, `$2`, `$3` | Positional arguments |
| `${10}` | Tenth argument and above |
| `$#` | Number of arguments |
| `$@` | All arguments |
| `$?` | Exit status of the last command |
| `$$` | Process ID of the current script |

Example:

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Number of arguments: $#"
```

---

## Variables

Variables store values for reuse.

Assignment uses no spaces around `=`.

```bash
target="10.10.10.1"
echo "Scanning $target..."
```

Incorrect:

```bash
target = "10.10.10.1"
```

Bash will treat `target` as a command and fail.

---

## Arrays

Arrays store multiple values in one variable.

```bash
domains=(www.example.com ftp.example.com mail.example.com)

echo ${domains[0]}
echo ${domains[@]}
```

Array indexes start at `0`.

If an array item contains spaces, quote it:

```bash
files=("Config File.txt" "Log Data.log")
```

---

## Conditional Logic

Conditional logic lets a script make decisions.

Basic structure:

```bash
if [ condition ]
then
    command
elif [ another_condition ]
then
    command
else
    command
fi
```

Example: validating that the user provided one argument.

```bash
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Error: You must specify a target domain."
    echo "Usage: $0 <domain>"
    exit 1
elif [ $# -eq 1 ]
then
    domain=$1
    echo "Processing domain: $domain"
else
    echo "Error: Too many arguments."
    exit 1
fi
```

---

## Comparison Operators

Bash uses different operators depending on whether you are comparing strings, numbers, or files.

### Integer Operators

| Operator | Meaning | Example |
|---|---|---|
| `-eq` | Equal to | `[ "$count" -eq 1 ]` |
| `-ne` | Not equal to | `[ "$count" -ne 0 ]` |
| `-lt` | Less than | `[ "$count" -lt 5 ]` |
| `-gt` | Greater than | `[ "$count" -gt 5 ]` |
| `-ge` | Greater than or equal to | `[ "$count" -ge 10 ]` |

### File Operators

| Operator | Meaning |
|---|---|
| `-e` | Path exists |
| `-f` | Is a regular file |
| `-d` | Is a directory |
| `-r` | Is readable |
| `-w` | Is writable |
| `-x` | Is executable |
| `-s` | File exists and is not empty |

Example:

```bash
#!/bin/bash

if [[ -e "$1" && -r "$1" ]]
then
    echo "Access granted: Reading $1..."
    cat "$1"
elif [[ -e "$1" && ! -r "$1" ]]
then
    echo "Permission error: You do not have read rights for $1."
    exit 1
else
    echo "Error: File not found."
    exit 2
fi
```

---

## Logical Operators

Use double brackets `[[ ... ]]` when combining conditions.

| Operator | Meaning |
|---|---|
| `&&` | Logical AND |
| `||` | Logical OR |
| `!` | Logical NOT |

Example:

```bash
if [[ -e "$file" && -r "$file" ]]
then
    cat "$file"
fi
```

---

## Arithmetic Operations

Bash supports integer arithmetic by default.

| Operator | Meaning |
|---|---|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulus |
| `++` | Increment |
| `--` | Decrement |

Use arithmetic expansion when you want to print or use the result:

```bash
echo "Total: $((10 + 5))"
```

Use arithmetic evaluation when updating a variable:

```bash
counter=0
((counter++))
echo "$counter"
```

Bash does not handle decimal math by default. Use tools such as `bc` for floating-point calculations.

---

## String Length

Use `${#variable}` to count the number of characters in a variable.

```bash
platform="HackTheBox"
echo "Length: ${#platform}"
```

---

## Functions

Functions make scripts easier to organize and reuse.

A function must be defined before it is called.

```bash
function greet {
    echo "Hello, $1!"
}

greet "John"
```

Alternative syntax:

```bash
greet() {
    echo "Hello, $1!"
}
```

Inside a function, `$1`, `$2`, and other positional parameters refer to the function arguments, not the script arguments.

---

## Variable Scope in Functions

By default, Bash variables are global.

Use `local` to limit a variable to a function.

```bash
calculate() {
    local result=50
    echo "$result"
}
```

This helps prevent accidental overwrites in larger scripts.

---

## Function Return Values

Bash functions return exit status codes, not normal data values.

| Return Code | Meaning |
|---|---|
| `0` | Success |
| Non-zero | Error or failure |

Example:

```bash
check_file() {
    if [[ -f "$1" ]]
    then
        return 0
    else
        return 1
    fi
}
```

To return actual text or data, print it with `echo` and capture it with command substitution:

```bash
get_date() {
    date +%F
}

today=$(get_date)
echo "Today is $today"
```

---

## Input with `read`

The `read` command lets a script accept user input.

```bash
read -p "Enter target: " target
echo "Target selected: $target"
```

The `-p` option displays a prompt before waiting for input.

---

## Case Statements

A `case` statement is useful for menus and handling multiple fixed options.

```bash
read -p "Select your option: " opt

case $opt in
    "1")
        echo "Running scan..."
        ;;
    "2")
        echo "Running exploit..."
        ;;
    *)
        echo "Invalid option."
        exit 1
        ;;
esac
```

Important syntax:

| Syntax | Meaning |
|---|---|
| `case $var in` | Starts the case block |
| `pattern)` | Matches a value |
| `;;` | Ends a branch |
| `*)` | Default catch-all branch |
| `esac` | Ends the case block |

---

## Loops

Loops repeat a block of code.

### For Loops

Use a `for` loop when you already have a known list.

```bash
for ip in 10.10.10.1 10.10.10.2 10.10.10.3
do
    ping -c 1 "$ip"
done
```

One-line version:

```bash
for ip in 10.10.10.1 10.10.10.2; do ping -c 1 "$ip"; done
```

### While Loops

A `while` loop runs while a condition is true.

```bash
counter=1

while [ $counter -le 5 ]
do
    echo "Counter: $counter"
    ((counter++))
done
```

### Until Loops

An `until` loop runs while a condition is false.

```bash
counter=0

until [ $counter -eq 10 ]
do
    ((counter++))
    echo "Counter is now: $counter"
done
```

### Loop Control

| Command | Meaning |
|---|---|
| `break` | Exit the loop immediately |
| `continue` | Skip the current iteration and move to the next one |

---

## Output Control with `tee`

Redirection saves output to a file but hides it from the terminal.

```bash
command > output.txt
```

The `tee` command displays output on the screen and saves it to a file at the same time.

```bash
command | tee output.txt
```

Append instead of overwrite:

```bash
command | tee -a output.txt
```

Comparison:

| Method | Syntax | Visible in Terminal | Saved to File |
|---|---|---|---|
| Standard output | `command` | Yes | No |
| Redirection | `command > file` | No | Yes |
| Tee | `command \| tee file` | Yes | Yes |

Example:

```bash
hosts=$(host "$domain" | grep "has address" | cut -d " " -f4 | tee discovered_hosts.txt)
```

This prints discovered hosts to the terminal and saves them to `discovered_hosts.txt`.

---

## Practical Script Example

This example combines arguments, validation, functions, loops, and output logging.

```bash
#!/bin/bash

if [ $# -ne 1 ]
then
    echo "Usage: $0 <domain>"
    exit 1
fi

domain=$1
output_file="discovered_hosts.txt"

find_hosts() {
    host "$domain" | grep "has address" | cut -d " " -f4 | tee "$output_file"
}

ping_hosts() {
    while read -r ip
    do
        if ping -c 1 "$ip" > /dev/null 2>&1
        then
            echo "$ip is up"
        else
            echo "$ip is down"
        fi
    done < "$output_file"
}

find_hosts
ping_hosts
```

---

## Quick Reference

| Concept | Syntax / Command |
|---|---|
| Shebang | `#!/bin/bash` |
| First argument | `$1` |
| Argument count | `$#` |
| All arguments | `$@` |
| Last exit status | `$?` |
| Variable assignment | `name="value"` |
| Array | `items=(one two three)` |
| If statement | `if [ condition ]; then ... fi` |
| Case statement | `case $var in ... esac` |
| For loop | `for item in list; do ... done` |
| While loop | `while [ condition ]; do ... done` |
| Function | `name() { commands; }` |
| Arithmetic | `$((x + y))` |
| Save and display output | `command \| tee file.txt` |