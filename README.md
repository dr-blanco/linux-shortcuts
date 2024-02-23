# linux-shortcuts

It is good to know how to start, enable, and view logs of a service. Here is a basic example of that.

```
systemctl start httpd
systemctl enable httpd
systemctl status httpd
journalctl -u httpd
```

Quick example of a python script and using the famous `perf` command to see its memory performance

```
cat << EOF > demo.py
import random
import time

def main():
    i=0
    numbers = random.choices(range(2, 250001), k=250000)
    print(
        sorted(
            [
                num for num in numbers
                if not any( (f:=[ True if num % i == 0 else False for i in range(2, int((num+1.0)**0.5))  ] ) )
            ]
        )[0:100]
    )

if __name__ == "__main__":
    main()
EOF
python3 demo.py > data.txt &
sudo perf stat -e page-faults -I 1000 -p $(pgrep -d ',' python)
```

Example of memory complexity and why Python benefits using generators 
```
cat << EOF > demo.py
import random
import sys

numbers = range(2, 100001)
print(sys.getsizeof(numbers)) # only 48 bytes, generators yield one value at a time instead of all of it in memory
print(sys.getsizeof(list(numbers)))
EOF
python3 demo.py
48
800048
```

Speaking of generators, we can easily make one using the yield keyword instead of return. Notice that the int type cast may round a number down, that is why we add one to it.
```
# Generator for prime numbers
def prime_gen(num_of_primes):
    num = 2
    while num_of_primes:
        is_prime = True
        for f in range(2, int(num ** 0.5) + 1):
            if num % f == 0:
                is_prime = False
                break
        if is_prime:
            num_of_primes -= 1
            yield num
        num += 1

# First 10 Prime numbers
for i in prime_gen(10):
    print(i)

```


Aggregating data with AWK can be fun (not too practical but fun indeed)
```
echo 'A 25 27 50
B 35 37 75
C 75 78 80
D 99 88 76' | awk '
{
 rowtotal=0
 for (i=2; i<=NF; i++){
   rowtotal+=$i 
 }
 avg=rowtotal/(NF-1)
 if (avg > 70){ # HAVING avg > 70
   printf "%s %.2f\n", $0, avg # SELECT *, AVG
 }
}'

C 75 78 80 77.67
D 99 88 76 87.67
```


Find all .sh files and make sure they are executable to all users
```
find test -name "*.sh" -exec sh -c 'chmod a+x {}' \;
```

My Favorite Linux Command is Xargs, since sed does not have a recursive option, I use xargs to edit multiple nested files at a time! Careful with the -i (inplace) option here. May be wise to use word boundaries \b.
```
grep -lr old_pattern | xargs sed -i 's/old_pattern/new_pattern/g' # this will replace old_pattern with new_pattern in all files
```

Nothing is more handy than regular expressions, whether its searching logs in LogQL, or old school grep with Perl
```
echo
echo "Please enter a password with 8 or more charaters, one lower case, upper case and numeric character."
read -s password
[[ `echo $password | grep -P "^(?=.*[a-z])(?=.*[0-9])(?=.*[A-Z]).{8,}$"` ]] && echo "Pass" || echo "Must contain one number, upper case, lower case, and min 8 characters" 
```
