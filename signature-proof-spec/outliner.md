merkle tree signature proofs outliner:

introduction:
- introduce the current 3 flavours of VCs reference kaliya young, “identity woman” paper
- propose an alternative to the 3 flavours (similar to Lightweight selective disclosure for verifiable documents on blockchain)
- rest of the paper will describe how it works
design of the proof:
- overview of credential
    - short example of a credential (simple example)
- explain how the data integrity works
    - talk about the salts (rainbow attacks)
    - normalising credential into individual key-pairs
    - hashing each of individual key-pairs
    - compute targetHash of credential
    - assembling of credential after calculating proof.
- selective disclosure
- merkle hash trees
    - short introduction
    - and how it affects current solution so far
limitations of the proof:
- data obfuscation limits
- salt size only grow linearly as more claims are added
    - might impact performance
related work:
- combining subtrees from different claims are not allowed unlike(reference related work Minimal Information Disclosure with Efficiently Verifiable Credentials)
- the set of claims itself is not a merkle tree unlike [reference A Linearly Homomorphic Signature Scheme From Weaker Assumptions]