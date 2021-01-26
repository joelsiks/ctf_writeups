
# Optimizer

Only information given is an ip and a port.

<hr>

Netcating the ip and the port we are prompted with:
```
you will be given a number of problems give me the least number of moves to solve them
level 1: tower of hanoi
[33, 58, 20, 32, 44, 38, 44, 45, 37, 18, 21, 64, 38, 10]
>
```

Since it seems trivial to parse the numbers as "towers" in a hanoi puzzle, we instead try to calculate the number of moves to solve a hanoi puzzle with the amount of towers that is equal to the length of the array we are given.

The minimal number of moves required to solve a Tower of Hanoi puzzle is 2^n − 1, where n is the number of disks. 

When enough prompts have been answered, a new prompt is shown:
```
level 2 : merge sort, give me the count of inversions
[53, 15, 30, 5, 1, 54, 62, 30, 58, 34, 47, 32, 51]
>
```

This prompt was more straight forward. Now we only need to calculate the count of inversions needed to sort the given array with merge sort.

When enough prompts have been solved, the flag is printed:

```
> you won here's your flag flag{g077a_0pt1m1ze_3m_@ll}
```

Since these calculations are tedious when done by hand, here is a simple python script that automate the calculations and print out the flag.

```python
$ cat solve.py

#!/usr/bin/python3

import socket
import re

# Source: https://gist.github.com/leonjza/f35a7252babdf77c8421
class Netcat:

    """ Python 'netcat like' module """

    def __init__(self, ip, port):

        self.buff = ""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.connect((ip, port))

    def read(self, length = 1024):

        """ Read 1024 bytes off the socket """

        return self.socket.recv(length)

    def read_until(self, data):

        """ Read data into the buffer until we have data """

        while not data in self.buff:
            self.buff += self.socket.recv(1024)

        pos = self.buff.find(data)
        rval = self.buff[:pos + len(data)]
        self.buff = self.buff[pos + len(data):]

        return rval

    def write(self, data):

        self.socket.send(data)

    def close(self):

        self.socket.close()

def merge_inversions(arr):

    n = len(arr)
    inv_count = 0
    for i in range(n):
        for j in range(i + 1, n):
            if (arr[i] > arr[j]):
                inv_count += 1

    return inv_count

def tower_of_hanoi_solves(n):
    return pow(2, n) - 1

if __name__ == '__main__':

    counter = 0
    nc = Netcat('207.180.200.166', 9660)

    while True:

        nc.buff = b''

        if len(nc.read_until(b"[").decode("utf-8")) == 0:
            nc.read_until(b"\n").decode("utf-8")

        nums_str = nc.read_until(b"]").decode("utf-8")
        nums_str = nums_str[:-1]

        nums = nums_str.split(", ")
        nums = list(map(int, nums))

        if counter < 25:
            nc.write(str.encode(str(tower_of_hanoi_solves(len(nums)))))
        else:
            nc.write(str.encode(str(merge_inversions(nums))))

        counter += 1
        if counter == 50:
            print(nc.read_until(b"}").decode("utf-8"))
            break

```