---
description: An incredible generative algorithm 
tags: generative algorithm pattern
---

# 1 - What does it do ? 



# 2 - The core algorithm 

From ....


```python

def run(...)

    propagate(wave, adj, periodic=periodic, onPropagate=onPropagate)
    try:
        pattern, i, j = observe(wave, locationHeuristic, patternHeuristic)
        if onChoice:
            onChoice(pattern, i, j)
        wave[:, i, j] = False
        wave[pattern, i, j] = True
        if onObserve:
            onObserve(wave)
        propagate(wave, adj, periodic=periodic, onPropagate=onPropagate)
        if wave.sum() > wave.shape[1] * wave.shape[2]:
            # return run(wave, adj, locationHeuristic, patternHeuristic, periodic, backtracking, onBacktrack)
            return run(
                wave,
                adj,
                locationHeuristic,
                patternHeuristic,
                periodic=periodic,
                backtracking=backtracking,
                onBacktrack=onBacktrack,
                onChoice=onChoice,
                onObserve=onObserve,
                onPropagate=onPropagate,
                checkFeasible=checkFeasible,
                depth=depth + 1,
                depth_limit=depth_limit,
            )
    except Contradiction:
    	...
```


```python
def propagate(wave, adj, periodic=False, onPropagate=None):
    last_count = wave.sum()

    while True:
        supports = {}
        if periodic:
            padded = numpy.pad(wave, ((0, 0), (1, 1), (1, 1)), mode="wrap")
        else:
            padded = numpy.pad(
                wave, ((0, 0), (1, 1), (1, 1)), mode="constant", constant_values=True
            )

        for d in adj:
            dx, dy = d
            shifted = padded[
                :, 1 + dx : 1 + wave.shape[1] + dx, 1 + dy : 1 + wave.shape[2] + dy
            ]
            # print(f"shifted: {shifted.shape} | adj[d]: {adj[d].shape} | d: {d}")
            # raise StopEarly
            # supports[d] = numpy.einsum('pwh,pq->qwh', shifted, adj[d]) > 0
            supports[d] = (adj[d] @ shifted.reshape(shifted.shape[0], -1)).reshape(
                shifted.shape
            ) > 0

        for d in adj:
            wave *= supports[d]

        if wave.sum() == last_count:
            break
        else:
            last_count = wave.sum()

    if onPropagate:
        onPropagate(wave)

    if (wave.sum(axis=0) == 0).any():
        raise Contradiction
``