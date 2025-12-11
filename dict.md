# Dictionaries in Python

> 笔者在研读 [Dictionaries and Python OOP](https://github.com/KEDIT2007/NJU-SICP-Essays/blob/main/dict-and-python-oop/Dictionaries%20and%20Python%20OOP.md) 的时候意识到自己对 Python 字典的实现了解不够，便在这一方向进行了探索。
> 我们的 [SICP](https://sicp.pascal-lab.net/2025/) 课程没有讲这一数据结构，我相信也有其他人想深入了解字典；我不妨写一份讲义，来仔细讲讲 Python 的字典，后来者也可少走些弯路。

> [!NOTE]
> 这份讲义不适合以下读者：
> 
> - 期望以此学习语法。
> - 期望以此提高课内成绩。
> 
> 建议在阅读后完成配套练习。

## Table of Scores

Considering we need a data structure to store every student's SICP scores.

We can solve this problem with list easily:

```python
students = [
    ["Axe", 100],
    ["Cola", 80],
    ["Sprite", 60]
]
```

### The Common Ways We Have Learnt

Given a student's name, to find his score, we can check every names **linearly** until a match is found, then we can know his index, and find out his score. 

```python
def search(name: str) -> Optional[int]:
    """
    Linear search in a unordered list
    """
    for student in students:
        if student[0] == name:
            return student[1]
    return None
```

Obviously, its time complexity is $O(n)$ (Linear). It implicates that if a table of student records grews from 100 to 100,000 records, the search time increases by a factor of 1,000. This scalability is unacceptable for applications that rely on massive lookups.

You can also use **binary search** to reduce the time complexity to $O(\log N)$ (Logarithmic), but the names must be sorted.

```python
def search(name: str) -> Optional[int]:
    """
    Binary search in a name-sorted list
    """
    left , right = 0, len(student) - 1
    while left <= right:
        mid = (left + right) // 2
        current_name = students[mid][0]
        if current_name == name:
            return students[mid][1]
        elif current_name < name:
            left = mid + 1
        else: # current_name > name
            right = mid - 1
    return None
```

This is excellent, for 1,000,000 entries it only requires about 20 comparisons. But does this fast enough?

We all know that, with trillions of data, we'd better reduce the time complexity to $O(1)$ (Constant), meaning the time taken is independent of the dataset size.

### The Mechanism of Constant Time Access

How is $O(1)$ achieved in computing? Only by **direct** calculation, specifically by using an index derived from the key itself.

Think of an ordinary array access. Find the element at a given index is $O(1)$ because the memory address is calculated **directly** from the base address and the given index.

Then the challenge came: the key (name) is not a direct numerical index, so we must use a "magical function" to map the key to a **unique** index, and this "magical function" should also be $O(1)$. The main logic of our new algorithm will like the following prototype.

```python
def search(name: str) -> Optional[int]:
    """
    Constant-time search using a "magical function"
    """
    index = magical_function(name)
    if students[index] is not None:
        return students[index][1]
    else:
        return None
```

In the next section we will talk about the Hash Function as the "magical funtion".

## Hash

Hashing is the idea of taking a value and being able to output a value that becomes a shortcut to it later.

For example, hashing apple may hash as a value of 1, and berry may be hashed as 2. Therefore, finding apple is as easy as asking the hash algorithm where apple is stored. While not ideal in terms of design, ultimately, putting all a’s in one bucket and b’s in another, this concept of bucketizing hashed values illustrates how you can use this concept: a hashed value can be used to shortcut finding such a value.

The key to achieving $O(1)$ access is to bypass searching entirely and instead use the key itself to calculate the item's location. This is the role of the **Hash Function** and the **Hash Table**.

### Hash Function

A **hash function** is an algorithm that reduces a larger value to something small and predictable, often a **fixed-size** integer, which we call the **hash value**. Generally, this function takes in an item you wish to add to your hash table, and returns an integer representing the array index in which the item should be placed.

<a name="example-hash-function"></a>For example, if our hash function is define as "the alphabetical index of the first letter in the string", if 0-indexed, then "Axe" will hash as 0, "Cola" will hash as 2, and "Sprite" will hash as 18.

> There are many hash functions in the world, and they can be categorized into two main types:
> 
> 1. **Non-Cryptographic Hash Functions**
>     - Goal: Speed and good distribution (low collisions).
>     - Examples: Python's `dict` Hash.
> 2. **Cryptographic Hash Functions**
>     - Goal: Security and collision resistance.
>     - Examples: MD5, SHA-1, SHA-256
>
> You can try a commonly used hash function in your Python terminal:
> 
> ```python
> >>> import hashlib
> >>> data = b"sicp"
> >>> hashlib.md5(data).hexdigest()
> '1c15252d5a6768b373bea27e8fa22a08'
> ```
>
> Such a big number! We can take its modulo to reduce it to a much smaller size.

### Hash Table

So, what is a hash table? Literally, it is a table with hash, or to speak more detailedly, a data structure like an array, storing keys and values.

For example, we can construct a 26-buckets array to store the scores of students, and we use [the alphabetical hash function mentioned before](#example-hash-function). Then, a hash table could be imagines as follows:

| index <br> (hash) | name <br> (key) | score <br> (value) | expression | 
| --: | --- | --- | --- |
| 0 | "Axe" | 100 | `ht[0] = 100` |
| 1 |
| 2 | "Cola" | 80 | `ht[2] = 80` |
| ... |
| 18 | "Sprite" | 60 | `ht[18] = 60` |
| ... |
| 25 |

Then, if you want to find out Sprite's score, you can simply call the hash function (it returns 18), then visit table[18] to fetch the data in a constant time.

It looks great, but what if we have another student "Salt" whose score is 90, then we called the hash function, and it returns 18, and we store into the table: `table[18] = 90`. Oh no, it **collides**.

### Hash Collision

So now we must solve the collision, and there are several ways.

People often use a linked list to solve the collision, but today we are talking about dictionaries in Python, so we will introduce Python's way to solve this problem. As for the linked list solution, it's your job.

> [!NOTE]
> Exercise 1: Hash Table with Linked List
> 
> Implemented a hash table with linked list, such that the collision will added to the end of the linked list.
> 
> ```python
> class Hashtable:
>     """
>     An implementation of Hashtable using Link.
> 
>     >>> scores = Hashtable()
>     >>> scores["Axe"] = 100
>     >>> scores["Axe"]
>     100
>     >>> scores["Sprite"] = 60
>     >>> scores["Sprite"]
>     60
>     """
>     def __init__(self, size=10):
>         self.size = size
>         self.buckets = [Link.empty] * size
>         self.count = 0
> 
>     def _hash_func(self, key):
>         """Hash Function"""
>         return hash(key) % self.size
>     
>     def __setitem__(self, key, value):
>         """Insert / Update"""
>         "*** YOUR CODE HERE ***"
> 
>     def __getitem__(self, key):
>         """Lookup"""
>         "*** YOUR CODE HERE ***"
> 
>         # Key not found
>         raise KeyError(key)
> 
>     def __len__(self):
>         return self.count
> ```
