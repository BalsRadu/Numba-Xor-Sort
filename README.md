# The Permutation Problem

**For more details, check the notebook!**

This is my solution to a relatively hard problem that was presented to me by a friend. While I didn't manage to solve it the first time, I recently encountered the problem again and decided to give it another shot.

The original solution to this problem was based on ordering your list of numbers in ascending order and then using the sum as a seed for a random number generator to shuffle the list. In the second function, you would verify all values for the possible element, insert it, sort the array, shuffle with the sum seed, remove the inserted element, and verify if it was equal to the given permutation. While this solution worked, it was quite inefficient and didn't quite feel right.

Initially, I inferred that we could embed some global information into the permutation itself, like a very naive error-correcting code. I didn't manage to find such an arrangement, but I knew it should, in theory, be possible. Another solution that I heard about—which I don't think would have worked in its initial formulation—was to use a hashing function for each element and then use the index position as the entry order in the permutation. There were two problems (disregarding the normal issues with hash functions):

- The hashing function can create "gaps" in the permutation space. For example, for a naive function _h(x) = x_, there are multiple initial permutations that, after removing the element, will yield the same ordering. For a given removed permutation [1, 2, 8, 9], there are multiple values that will produce the same ordering—[1, 2, 3, 8, 9], [1, 2, 4, 8, 9], [1, 2, 5, 8, 9], and so on. This means the proposed solution is **NOT LIST DEPENDENT**, or in other words, the sorting of the elements depends only on _x_, not also on a global property _P_ of the list of elements (this can also be seen in the original solution, where the seed is the sum of the list). The hashing ordering and the sum seed shuffling are both ways of ordering the elements based on a global property of the list (like a checksum). **THIS IS A WAY TO EMBED INFORMATION INTO THE PERMUTATION**, but it is not directly recoverable from the structure as in the case of error-correcting codes (I don't think you can recover the missing number this way without adding additional elements or modifying the existing ones), **BUT FROM THE PROCESS OF GENERATING THE PERMUTATION!** This is viable **ONLY WHEN** the range of values is known and is not too big to become impractical.

- The hashing itself depends on the ordering in which the elements were given. In an ideal case, this function would depend only on the list of elements, regardless of their order. This can be solved by putting the originally generated list of elements into a deterministic permutation, such that we eliminate the dependency on the ordering in the hash functions. **THIS IS SUFFICIENT TO SOLVE THE PROBLEM WITH A HASH METHOD!** (barring any other issues that might arise from the hashing function itself, such as collisions and so on). We also see this property in the original solution, where for the shuffling to work, the list of elements must be ordered in ascending order.

Here we can also start to postulate that all the solutions to this problem are related and are part of the same family of solutions. Not only that, but any solution to this problem **MUST** satisfy the above conditions. Given these observations, I came up with an alternative solution that directly sorts the elements of the list based on the expression _x ^ S_ (this satisfies both property 1 and 2). While it was neat that we found a different solution to this hard problem, it still was not much of an improvement in terms of performance. The solution was still _O(max_int \* N log N)_ and mostly used native Python. This approach won't solve the problem in a reasonable amount of time for any relatively large range of values or list of elements.

After a lot of code restructuring and optimization, I managed to achieve a final solution of _O(parallelizable(max_int) \* N)_. This is the result of optimizations on multiple levels:

- **Algorithmic:** I used localized binary search, short-circuiting, and other techniques to reduce the number of operations needed to find the missing element.
- **Language:** I made use of NumPy and Numba to speed up the operations, vectorize tasks, and reduce the overhead of the Python interpreter. We also employed memory optimizations such as using contiguous arrays and performing operations in place.
- **Parallelization:** Since the search over the possible values of the missing number is independent of the process of verifying whether it is the correct one, we can parallelize it. I divided the search space into multiple parts and then used Joblib to launch multiple processes to search for the missing number in chunks.

This solution is very efficient and works reasonably fast for any _N_ and _max_int <= 10^6_. While I don't think this solution can be improved much further in terms of implementation and algorithm, there are still some things that could speed it up:

- Translate this solution to a low-level language like C++.
- Try to run this solution on a GPU.
- Dynamic finding of the optimal chunk size for parallelization.
- Further optimizations across the board.
