### operators
- is, is not: they are used to check if both the reference points to the same object in the memory

### boolean vs logical operators
-	and, or, not → for booleans/logical conditions
-	&, |, ^, ~, <<, >> → for bit-level manipulation of integers

### type conversion in python
- float(res)
- int(res)
- str(res)
- bool(res)
- complex(res) (complex number)

### user input
- sname = input("Enter name")
- age = int(input("Enter age"))

### data types
- 5500L for long data types

### strings
```python
txt = "python"

print(txt[0:2])    # 'py' → end (2) not included
print(txt[-1])     # 'n'  → -1 is last char
print(txt[-2])     # 'o'  → -2 is 2nd last char
print(txt[0:-1])   # 'pytho' → slice till last-1
print(txt[::-1])   # 'nohtyp' → reverse (step = -1)
print(txt[::2])    # 'pto' → take every 2nd char
print(txt[::-2])   # 'nhy' → take every 2nd char backwards
```

### lambda function

```python
# Lambda function examples

add = lambda x, y: x + y
print(add(3, 5))         # 8 → sum of two numbers

square = lambda x: x**2
print(square(4))         # 16 → square of a number

reverse = lambda s: s[::-1]
print(reverse("python")) # 'nohtyp' → reverse string

even = lambda x: x % 2 == 0
print(even(6))           # True → check even

starts_p = lambda s: s.startswith('p')
print(starts_p("python")) # True → check prefix

max_val = lambda a, b: a if a > b else b
print(max_val(10, 20))   # 20 → max of two numbers

words = ["apple", "banana", "cherry"]
print(sorted(words, key=lambda w: len(w))) 
# ['apple','banana','cherry'] → sort by length

nums = [1,2,3,4,5]
print(list(map(lambda x: x*2, nums)))   
# [2,4,6,8,10] → double each element

print(list(filter(lambda x: x%2, nums)))  
# [1,3,5] → keep only odd numbers

from functools import reduce
print(reduce(lambda a,b: a*b, nums))    
# 120 → product of all numbers
```

### map and filter
```python
# 1) Square only even numbers
nums = [1, 2, 3, 4, 5, 6]

# filter: (function, iterable)
evens = filter(lambda x: x%2==0, nums)   # keep evens
squares = map(lambda x: x**2, evens)     # square them
print(list(squares))  
# [4, 16, 36]

# 2) Convert input (comma separated) → list of ints
data = "10,20,30,40"
nums = list(map(int, data.split(",")))
print(nums)  
# [10, 20, 30, 40]

# 3) Input space separated → keep only > 50
data = "23 67 12 89 45"
nums = list(map(int, data.split()))
big = list(filter(lambda x: x>50, nums))
print(big)  
# [67, 89]

# 4) Convert strings → uppercase only if length > 3
words = ["hi", "python", "is", "fun"]
res = list(map(lambda w: w.upper(), filter(lambda w: len(w)>3, words)))
print(res)  
# ['PYTHON']

# 5) Add 5 to odd numbers only
nums = [1, 2, 3, 4, 5]
res = list(map(lambda x: x+5, filter(lambda x: x%2, nums)))
print(res)  
# [6, 8, 10]
```

```python

v, e = map(int, input().split())
adj = ([0] * v for _ in range(v))

for k in range(e)
  i, j = map(int, input.split())
  adj[i][j] = 1
  adj[j][i] = 1

count = 0
for i in range(v):
  for j in range(i + 1, v):
    for k in range(j + 1, v):
      if adj[i][j] == 1 and adj[j][k] == 1 and adj[k][i] == 1
        count += 1

print(count)

# p = l.strip().split('|', 1)
# set: len(c), c.clear(), c.add(1), c.discard(1)
# Use - 
# v in c and c.discard(v) is None (if discard was successful)
# v not in c and c.add(v) is None (if add was successful)
```

### ds
```python
myset = set()
myset.add(1)

mylist = [1, 2, 1]
myset = set(mylist)

s = set()
if i in s # check value in set
s.add(2)
s.clear()
s.discard(2) # removes
sc = s.copy()
s.difference(sc) # returns difference between 2 sets
s1.difference_update(s2) # removes common el btw s1 and s2 from s1
```