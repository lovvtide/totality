# totality

### What is it?

This module is my zero-dependency reference implementation for an algormithm that computes a numerical rank for each node in a web of trust. It's implemented abstractly so it can in principle be used in any context where nodes "vouch" for each other in some way.

### How does it work?

You run it by passing it a big array of json objects that each have a "source" and a "target". In the case of nostr's web of trust this would be the "following", and "followed" pubkey, respectively.

There starting point conceptually is the idea that every node in a web of trust accrues "rank" not only by direct follows, but also by follows-of-follows (follows-of-follows-of-follows, and follows-of-follows-of-follows-of-follows, ad infinitum). But all these follows are not the same: direct follows are more important.

So the algorithm weights these more distant "vouches" according to a logarithmic decay, and by default cuts off computation at six degrees of separation (this can be modified by the `maxDegrees` option). The rate of the logarithmic decay is modulated by a parameter `lambda` which is a measure of the average connectivity of the web of trust which helps to normalize the score across netorks that are "clumpy" in the sense that a few nodes have a much larger number of direct follows than most (which is most networks, since follow counts are Pareto distributed)

The last point is that we only look at the MINIMUM degree of separation. There may me many "paths" through a web of trust between any two nodes. The algorithm only considers the shortest with the highest contribution to a node's rank and tosses the rest out.

Mathematically, you can express the rank of each node as for each node as:

Rank(i) = Σ (1 / λ^d(i,j))

where

- j represents each source connected to i,
- d(i,j) is the degree of separation between i and j
- λ is the total count of sources _that are also targets_, divided by the total number of nodes

The function always returns an object like `{ node, tree, rank, lambda }`

`node` is a flat mapping of nodes by their `id` (the thing they're referred to as in the source/target of links)
`tree` is all the nodes in a tree reflecting the structure of the network that can be iterated over recursively
`rank` is a flat mapping of each node by its `id` to the computed "trust" score (whatever that means in context)
`lambda` is the average connectivity parameter

### But where does the scarcity come from?

That's out of scope—the algoritm doesn't specifiy.

For the nostr web-of-trust use case, the source of scarity would be the user's social graph.

The obvious use case is the kind 3 follow likst, but I think this may be useful for modulating search results (with the source and target being constructed by events that reference other events) and also [NIP-29](https://github.com/nostr-protocol/nips/blob/master/29.md) communites (where source is a user's pubkey, and the target is the pubkey of the admin of a group the user has participated in)

There may be other use cases but those are the ones I'm working on.
