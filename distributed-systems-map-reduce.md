# Map Reduce

Originally designed, built and used by Google. [Paper](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf). Wanted to be allow engineers who aren't experts on Distributed Systems to be able to work scale their software out to thousands of machines. Programmers write a "map" function and a "reduce" function, the system takes care of the distribution and scaling.

#### Approach

- Take some giant input, and split it in to many small input files.
  - input 1: foo, bar, fizz
  - input 2: bar, foo
  - input 3: foo
- Define a map function and execute it on each of the input files.
  - e.g. Word count, map function emits words with their counts.
  - Gives an intermediate output which is a set of (key, value) pairs.
  - input 1: (foo, 1), (bar, 1), (fizz, 1)
  - input 2: (bar, 1), (foo, 1)
  - input 3: (foo, 1)
- Define a reduce function
  - Reducer is given all of the values (v) of a given key (k).
  - e.g. From word count
    - [(foo, 1), (foo, 1), (foo, 1)]
    - [(bar, 1), (bar, 1)]
    - [(fizz, 1)]
- Create some output or aggregation via the reducer.

- Example map algorithm
```
Map(k, v)
  split k in to words
  for each word w
    emit(k, 1)
```

- Example reduce algorithm
```
Reduce(k, v[])
  emit(length(v[]))
```

- Example result
  - (foo, 3)
  - (bar, 2)
  - (fizz, 1)

#### Limitations

- Network throughput could cause some issues.
  - In Google's case, they only had roughly 50mbit/s available for each machine in their cluster.
    - To get around this, set up GFS on the same machines that did the map functions.
  - Stored the result of the map on the local disk of the machine that did the map.
  - Networks have evolved since 2004, and are much faster.
    - No longer the case, for the most part.