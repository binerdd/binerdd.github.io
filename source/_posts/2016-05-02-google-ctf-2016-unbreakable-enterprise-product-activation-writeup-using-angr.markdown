---
layout: post
title: "google ctf 2016 unbreakable enterprise product activation writeup (using angr)"
date: 2016-05-02 22:51:15 +0900
comments: true
categories: [ctf, googlectf, reversing] 
---

This was a simple binary that accepts product key as a commandline argument and it prints `Thank you - product activated!` message if the key is correct.

{% img center /images/googlectf-unbreakable-1.png %}

Loading the binary in IDA, you can see a basic check is made whether commandline argument is given and branches to the right basic block if true. The commandline argument, which is the product key given by the user, is copied to a global buffer `dest` by the size `0x43` using `strncpy`. And rest of the code calls all these functions that do byte operations on the product key and terminate if key is invalid. If the given product key passes all the byte operation checks, at last `ActivationSuccess` function is called.

{% img center /images/googlectf-unbreakable-2.png %}
{% img center /images/googlectf-unbreakable-3.png %}

One can reverse engineer all the checks done inside the functions and generate the product key, but there are too much! Therefore instead of going through all the functions, I decided to use `angr`. Symbolic execution is good at this type of "find satisfying key" type challenge. 

Here is the python script for solving the challenge with `angr`.

```python
import angr
import claripy
import simuvex

angr.path_group.l.setLevel( "DEBUG")

p = angr.Project( "unbreakable-enterprise-product-activation" , load_options={'auto_load_libs' : False})

flag = claripy.BVS( 'flag',0x43 * 8)
initial_state = p.factory.blank_state(addr= 0x4005BD)
initial_state.mem[ 0x6042c0] = flag

for b in flag.chop(8):
        initial_state.add_constraints(b!= 0)

pg = p.factory.path(initial_state)

ex = p.surveyors.Explorer(start=pg, find= 0x400830)
ex.run()
state = ex.found[ 0].state

for i in range( 8):
        b = state.memory.load( 0x6042C0+i,1 )
        state.add_constraints(b>= 0x21, b<=0x7E )

print hex(state.se.any_int(state.memory.load(0x6042C0, 0x43)))[2 :-1].decode( 'hex').strip(' \0')

```

I started analysis at the address right after the `strncpy`, so that I can make the global buffer `dest` symbolic without worrying about passing symbolic string as commandline argument, etc. `0x4005BD` is the address right after the `strncpy` and `0x6042C0` is the address of the global buffer. From the code, you can see that I made a symbolic string of size `0x43` and stored it to the global buffer. After that few constraints are added to make sure there are no null bytes and are printable ascii bytes(actually got the contraint code from other angr [writeup](https://github.com/angr/angr-doc/blob/master/examples/whitehat_crypto400/solve.py)).

Then all you have to do is find a path to `0x400830` using `Surveyors` which is the address of the call instruction to `ActivateSuccess` function.
Finding the satisfying symbolic string produces the key:

```bash
(angr) vagrant@irene64:/vagrant$ python unbreakable.py
CTF{0The1Quick2Brown3Fox4Jumped5Over6The7Lazy8Fox9}@@@@@@@@@@@@@@@@
```
