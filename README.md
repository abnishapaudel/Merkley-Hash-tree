# Merkley-Hash-tree
This contains the basic information along with the implementation using JAVA

# Introduction to Merkley tree/Hash tree
Merkle tree also known as hash tree is a data structure used for data verification and synchronization. It is a tree data structure where each non-leaf node is a hash of it’s child nodes. All the leaf nodes are at the same depth and are as far left as possible. It maintains data integrity and uses hash functions for this purpose. 

# Hash Functions:  
A hash function maps an input to a fixed output and this output is called hash. The output is unique for every input and this enables fingerprinting of data. So, huge amounts of data can be easily identified through their hash. 

![image](https://user-images.githubusercontent.com/104343178/165032254-da822487-98f5-4da9-9e5a-a9650a825cc4.png)<br/>
This is a binary merkel tree, the top hash is a hash of the entire tree. 
 

# For a Binary Merkel tree 
 
# Operation	     Complexity
Space	         O(n) </br>
Searching 	     O(logn) </br>
Traversal	     O(n)</br>
Insertion	     O(logn)</br>
Deletion	     O(logn)</br>
Synchronization	 O(logn)</br>

# Implementation of Merkley tree
![image](https://user-images.githubusercontent.com/104343178/165032687-b285c496-6a69-4913-974a-564bcf4050ba.png)</br>
a, b, c, and d are some data elements (files, public/private keys, JSON, etc) and H is a hash function. If you’re unfamiliar, a hash function acts as a “digital fingerprint” of some piece of data by mapping it to a simple string with a low probability that any other piece of data will map to the same string. Each node is created by hashing the concatenation of its “parents” in the tree.</br>
The tree can be constructed by taking nodes at the same height, concatenating their values, and hashing the result until the root is reached. A special case needs handled when only one node remains before the tree is complete, but other than that the tree construction is somewhat straightforward (more on this in the implementation section).</br>
Once built, data can be audited using only the root hash in logarithmic time to the number of leaves (this is also known as a Merkle-Proof). Auditing works by recreating the branch containing the piece of data from the root to the piece of data being audited. In the example above, if we wanted to audit c (assuming we have the root hash), we would need to be given H(d) and H(H(a) + H(b)). We would hash c to get H(c), then concatenate and hash H(c) with H(d), then concatenate and hash the result of that with H(H(a) + H(b)). If the result was the same string as the root hash, it would imply that c is truly a part of the data in the Merkle Tree.</br></br>
In a case such as torrenting, another peer would provide the piece of data, c, H(d), and H(H(a) + H(b)). If you’re concerned about the security of this approach, recall that in a hash function it is computationally infeasible find some e such that H(e) = H(c). This means that so long as the root hash is correct, it would be difficult for adversaries to lie about the data they were providing.</br>
Outputting the authentication path of some data is as simple as recreating the branch leading up until the root. Traversing the entire tree to produce the leaves and their respective authentication data becomes important when using the Merkle Tree in digital signature schemes, and this can actually be accomplished in under logarithmic time.</br>
</br>



# Assumptions taken while implementing the Merkley tree
</br>
The THEX Merkle Tree design was the inspiration for my implementation, but for my use case I made some simplifying assumptions. For one, I start with the leaves already having a signature. Since THEX is designed for file integrity comparisons, it assumes that you have segmented a file into fixed size chunks. That is not the use case I'm targeting.The THEX algorithm "salts" the hash functions in order to ensure that there will be no collisions between the leaf hashes and the internal node hashes. It concatenates the byte 0x01 to the internal hash and the byte 0x00 to the leaf hash:</br>
internal hash function = IH(X) = H(0x01, X)</br>
leaf hash function = LH(X) = H(0x00, X)</br>
It is useful to be able to distinguish leaf from internal nodes (especially when deserializing), so I morphed this idea into one where each Node has a type byte -- 0x01 identifies an internal node and 0x00 identifies a leaf node. This way I can leave the incoming leaf hashes intact for easier comparison by the downstream consumer.</br>
</br>

# So my MerkleTree.Node class is:</br>
static class Node {</br>
  public byte type;  // INTERNAL_SIG_TYPE or LEAF_SIG_TYPE</br>
  public byte[] sig; // signature of the node</br>
  public Node left;</br>
  public Node right;</br>
}</br>

# Hash/Digest Algorithm </br>
Since the leaf nodes are being passed in, my MerkleTree does not know (or need to know) what hashing algorithm was used on the leaves. Instead it only concerns itself with the internal leaf node digest algorithm. The choice of hashing or digest algorithm is important, depending if you want to maximize performance or security. If one is using a Merkle tree to ensure integrity of data between peers that should not trust one another, then security is paramount and a cryptographically secure hash, such as SHA-256, Tiger, or SHA-3 should be used.</br>
For my use case, I was not concerned with detecting malicious tampering. I only need to detect data loss or reordering, and have as little impact on overall throughput as possible. For that I can use a CRC rather than a full hashing algorithm. Earlier I ran some benchmarks comparing the speed of Java implementations of SHA-1, Guava's Murmur hash, CRC32 and Adler32. Adler32 (java.util.zip.Adler32) was the fastest of the bunch. The typical use case for the Adler CRC is to detect data transmission errors. It trades off reliability for speed, so it is the weakest choice, but I deemed it sufficient to detect the sort of error I was concerned with.</br>

So in my implementation the Adler32 checksum is hard-coded into the codebase. But if you want to change that we can either make the internal digest algorithm injectable or configurable or you can just copy the code and change it to use the algorithm you want.</br>
The rest of the code is written to be agnostic of the hashing algorithm - all it deals with are the bytes of the signature.


# References
https://www.geeksforgeeks.org/introduction-to-merkle-tree/</br>
https://www.javatpoint.com/blockchain-merkle-tree</br>
https://en.wikipedia.org/wiki/Merkle_tree</br>


