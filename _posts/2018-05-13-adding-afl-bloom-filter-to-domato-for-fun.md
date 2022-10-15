---
layout: post
title: "Adding AFL Bloom Filter to Domato for Fun"
date: 2018-05-13
---

So I have been playing with Domato for few weeks now and wrote a blog about it's internals along the way. Unfortunately there is no feedback mechanism in the open-sourced version of Domato, although according to the blog post by Ivan Fratic (@ifsecure) of google project zero they are experimenting with coverage guided fuzzing using modified version of WinAFL's DynamoRIO client. I had a chat with Richard Johnson (@richinseattle) of Cisco Talos Vulndev Team while staying in Dubai for OPCDE 2018 and he gave me an idea of adding some kind of feedback mechanism in the syntax level instead of binary level feedback. First thing that came up to my mind was AFL's bloom filter. Since it is known to be so effective, why not try adding this to Domato as well?

# Adding unique IDs to grammar rules

If you read my previous post, you will have understanding of how grammar rules are parsed and stored in data structures. In AFL, unique ids are assigned to each basic block. In our case, assigning unique ids to each grammar rules seems appropriate since order of selecting certain grammar rules for expanding symbols is similar to visiting basic blocks in the control flow graph. 

So in the grammar.py file of Domato, I made following changes:

```python
def _parse_code_line(self, line, helper_lines=False):
    """Parses a rule for generating code."""
    rule = {
        'type': 'code',
        'parts': [],
        'creates': [],
        'uid' : self._get_new_uid() # add this field
    }

def _parse_grammar_line(self, line):
    """Parses a grammar rule."""
    # Check if the line matches grammar rule pattern (<tagname> = ...).
    match = re.match(r'^<([^>]*)>\s*=\s*(.*)$', line)
    if not match:
        raise GrammarError('Error parsing rule ' + line)

    # Parse the line to create a grammar rule.
    rule = {
        'type': 'grammar',
        'creates': self._parse_tag_and_attributes(match.group(1)),
        'parts': [],
        'uid': self._get_new_uid() # add this field
    }
```

Now each rule contains a field named 'uid' that stores unique integer value. 

# Selecting less chosen rule for expanding symbols

To record which rules were chosen in the previous generated case, I added a Python dictionary for this purpose. 

```python
# coverage map
self._cov = {}
```
Now when choosing the rule for expanding current symbol, we need to reference the coverage map. Note that algorithm used in AFL is like following.

```python
cur_location = <COMPILE_TIME_RANDOM>;
shared_mem[cur_location ^ prev_location]++;
prev_location = cur_location >> 1;
```

So in our case,  cur_location is the unique id of a rule we are considering to select. prev_location is previously chosen rule. Following code checks whether if there is an never selected order of grammar rules (thus bitmap_idx is not in the coverage map) and selects first encountered rule of this case.

If every combination of previously selected grammar rule and current rule candidates have been selected before (so every possible bitmap_idx is already present as a key in coverage map), code saves how many times certain combination has been chosen.

```python
def _select_creator(self, symbol, recursion_depth, force_nonrecursive):
     ...
     # select creator based on coverage map
     # select creator that has been selected less
     hits = []
     for c in creators:
         bitmap_idx = (c['uid'] ^ self._prev_id) % MAP_SIZE
         if bitmap_idx not in self._cov:
             self._cov[bitmap_idx] = 1
             self._prev_id = c['uid'] >> 1
             return c
         else:
             hits.append(self._cov[bitmap_idx])
```

Among the grammar rules chosen, code saves a list (less_visited_creators[]) of relatively least selected rules. If cdf is not available for rules for some symbol, it randomly chooses from less_visited_creators[] and records coverage info accordingly.


```python
idx = random.randint(0, len(less_visited_creators) - 1)
curr_id = less_visited_creators[idx]['uid']
self._cov[(self._prev_id ^ curr_id) % MAP_SIZE] = self._cov[(self._prev_id ^ curr_id) % MAP_SIZE] + 1
self._prev_id = curr_id >> 1
return less_visited_creators[idx]
```

Similarly, if cdf is available for some symbol, it uses cdf instead randomly choosing a grammar rule and records coverage information.

```python
idx = bisect.bisect_left(less_visited_creators_cdf, random.random(), 0, len(less_visited_creators_cdf)-1)
curr_id = less_visited_creators[idx]['uid']
self._cov[(self._prev_id ^ curr_id) % MAP_SIZE] = self._cov[(self._prev_id ^ curr_id) % MAP_SIZE] + 1
self._prev_id = curr_id >> 1
return less_visited_creators[idx]
```

# Results

Because I lack resources to be used for fuzzing, testing was somewhat limited. Also I believe Google fuzzed modern browsers with Domato (or customized internal versions) long enough so there is a very low chance of myself finding exploitable crashes with my limited resources. However, my version of Domato was able to find more unique crashes than the original open-sourced version after fuzzing IE11 on Windows 10.

I generated 10,000 html files with both original Domato and my customized version and used them to fuzz IE11 to see which gets more unique crashes. The result if the following.

Original : 16 unique crashes
Customized : 20 unique crashes
I will soon release the code for those who are interested (if there is someone :P), although I guess by reading this blog you have enough information to go try out by yourself.

Any feedbacks or reporting of errors, please reach out ! 

