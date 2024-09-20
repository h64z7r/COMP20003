java cAssignment 2: Spellcheck Lookup
0. General
You must read fully and carefully the assignment speciﬁcation and instructions.
Course: COMP20003 Algorithms and Data Structures @ Semester 2, 2024
Deadline Submission: Friday 6th September 2024 @ 11:59 pm (end of Week 7)
Course Weight: 15%
Assignment type: individual
ILOs covered: 1, 2, 3, 4
Submission method: via ED
Purpose
The purpose of this assignment is for you to:
Improve your proﬁciency in C programming and your dexterity with dynamic memory 
allocation.
Demonstrate understanding of a concrete data structure (radix tree) and implement a set of 
algorithms. 
Practice multi-ﬁle programming and improve your proﬁciency in using UNIX utilities.
Practice writing reports which systematically compare the eﬃciency of diﬀerent data structures.
Walkthrough1. Background
In Assignment 1 you implemented a dictionary using a linked list. Many applications of dictionaries 
cannot assume that the user will query in the correct format. Take Google search as an example. If 
someone is so excited to check the recent Olympics updates, they may accidentally search something 
similar to this: 
Thankfully, there is enough context in the original query that the intended search result is 
recommended back to the user. In Assignment 2 you will be extending your implementation to 
account for misspelled keys when querying your dictionary.2. Your Task
Assignment:
Overall, you will create a partial error-handling dictionary (spellchecker) using a radix tree. 
You will be using the same dataset as Assignment 1. 
Users will be able to query the radix tree and will get either the expected key, or the closest 
recommended key.
You will then write a report to analyse the time and memory complexity of your Assignment 1 
linked list compared to your radix tree implementation. 
C Implementation:
Your programs will build the dictionary by reading data from a ﬁle. They will insert each 
suburb into the dictionary (either the linked list (Stage 3) or radix tree (Stage 4)).
Your programs will handle the search for keys. There are three situations that your programs 
must handle:
Handle exact matches: output all records that match the key (Stage 3 and 4).
Handle similar matches: if there are no exact matches, ﬁnd the most similar key and 
output its associated records (Stage 4 only).
Handle no matches being found: if neither exact nor similar matches are found, indicate 
that there is no match or recommendation (Stage 3 and 4). 
Your programs should also be able to populate and query the dictionary with the Assignment 
1 linked list approach and the radix tree approach. 
Your programs should also provide enough information to compare the eﬃciency of the 
linked list with the radix tree with a operation-based measure that ignores less important runtime
 factors (e.g. comparison count).
It is recommended to use your own solution of Assignment 1, and extend it for Assignment 2. But, we will also 
provide a solution for Assignment 1 if you prefer to work with our code.3. An Introduction to Tries and Radix Trees
First, it is important to establish the diﬀerence between a Trie and a Tree. This is best illustrated 
with an example. One example of a tree is a binary search tree (BST), where each node in the tree 
stores an entire string, as illustrated below. The nodes are ordered and allow easy searching. When 
searching in the BST, we compare the query with the entire string at each node, deciding whether to 
switch to the left or right subtree or stop (if the subtree is empty) based on the result of the 
comparison.
A trie is slightly diﬀerent. It has multiple names, such as retrieval tree or preﬁx tree. In a trie, the 
traversal of the tree determines the corresponding key. For the same strings as above with one 
letter per node, it would look like:Tries allow for quick lookup of strings. Querying this trie with the key "hey" would ﬁnd no valid path 
after the "e" node. Therefore, you can determine that the key "hey" does not exist in this trie.
A radix trie is a subtype of trie, sometimes called a compressed trie. Radix trees do not store a single 
letter at each node, but compress multiple letters into a single node to save space. At the character 
level, it would look like this:
Radix tries can again be broken down into diﬀerent types depending on how many bits are used to 
deﬁne the branching factor. In this case, we are using a radix (r) of 2, which means every node has 2 
children. This is accomplished by using 1 bit of precision, so each branch would be either a 0 or 1. 
This type of radix trie (with r=2) is called a Patricia trie. Another example of a radix trie with r=4 
would have 4 children, determined by the binary numbers 00, 01, 10, 11. The radix is related to the 
binary precision by r = 2x where x is the number of bits used for branching.
Radix trees beneﬁt from less memory usage and quicker search times when compared to a regular 
trie.
While these visual representations are at the character level, a Patricia trie is implemented using 
the binary of each character. Each node in the trie stores the binary preﬁx at the current node, 
rather than the character preﬁx. For example, we can insert 3 binary numbers into a PATRICIA trie: 
0000, 0010 and 0011.Combining the previous worded example with the binary representation gives us a patricia tree of the 
form:
You should trace along each path and validate that the stored strings are the same as the example 
above. Each character is represented over 8 bits, and the last character is followed by an end of string 
00000000 8 bit character, i.e. NULL .
It is important to note that these representations only show the preﬁx at each node. An actual 
implementation will require more information within a node struct. To see this, look at the "extra 
insertion example" slide.
Implementation TipsTo implement the Radix Tree, a recommended approach is to use the functions provided below.
1. Get a particular bit from a given character getBit(key, n) and 
2. Extract a stem from a given key splitStem(key, start, length) which extracts a certain 
number of bits from a given key. 
If you want to understand the code provided, you need to understand the following bit operators 
over numbers a and b:
 
Operator
a  b
a | b
a |= b
a > b
Meaning
Where both corresponding bits in a and b are 1, 
give a 1 in the output, else give a 0 in that position
Where either corresponding bit in a and b is 1,
give a 1 in the output, else give a 0 in that position. 
Compute a | b and assign the result of the operation to a.
For all bits in a, move each bit b bits to the left.
For all bits in a, move each bit b bits to the right.
Use The Following Functions
The two recommended functions, getBit and splitStem are provided here as they can typically be 
used without making major changes to their structure. Diagrams explaining their algorithmic 
structure and example outputs are also provided.
getBit 
This function works out the byte which the bit we're trying to extract lies in and then works out which 
bit it will be in. The oﬀset is not simply the remainder of the division because the bits are ﬁlled and 
split from the left (highest value bit) rather than the right.
/* Number of bits in a single character. */
#define BITS_PER_BYTE 8
/* Helper function. Gets the bit at bitIndex from the string s. */
static int getBit(char *s, unsigned int bitIndex);
static int getBit(char *s, unsigned int bitIndex){
 assert(s  bitIndex >= 0);
 unsigned int byte = bitIndex / BITS_PER_BYTE;
 unsigned int indexFromLeft = bitIndex % BITS_PER_BYTE;
 /* 
 Since we split from the highest order bit first, the bit we are interested
 will be the highest order bit, rather than a bit that occurs at the end of the
 number. 
 */
 unsigned int offset = (BITS_PER_BYTE - (indexFromLeft) - 1) % BITS_PER_BYTE;
 unsigned char byteOfInterest = s[byte]; unsigned int offsetMask = (1 > offset;
 return bitOnly;
}
An example diagram of what the calculation of the offset does from each possible value of 
indexFromLeft is shown here:
Oﬀset
7 indexFromLeft
0
6
1
5
2
4
3
3
4
2
5
1
6
0
7 oﬀset
An example for getting bit number 12 from the string "hi" could be called using getBit("hi", 12) . 
An example of how the calculation occurs is shown below:getBit
Input
IndexFromLeft
Oﬀset
byteOfInterest
oﬀsetMask
maskedByte
 bitOnly
s
bitIndex
0b01101000 01101001 00000000
h i \0
s[byte]
12
bitIndex % 8 bitIndex / 8
4
4 7 indexFromLeft
0
6
1
5
3 2
3
4
2
5
1
6
0
7 oﬀset
1 > oﬀset
byte = 1
0 1 1 0 1 0 0 1
byteOfInterest  oﬀsetMask
0 0 0 0 0 0 0 1
0 0 0 0 1 0 0 0代 写COMP20003、c/c++，Java
代做程序编程语言
0 0 0 0 1 0 0 0
0 0 0 0 0 0 0 1
createStem
This function creates a new collection of bytes from an existing collection of bytes. Conceptually this 
is extracting a substring from a string and returning the new substring, but the returned value will not 
necessarily be NULL terminated and may not start from the beginning of a character - so may not be 
printable as a string for multiple reasons.
/* Allocates new memory to hold the numBits specified and fills the allocated
 memory with the numBits specified starting from the startBit of the oldKey
 array of bytes. */
char *createStem(char *oldKey, unsigned int startBit, unsigned int numBits);
char *createStem(char *oldKey, unsigned int startBit, unsigned int numBits){
 assert(numBits > 0  startBit >= 0  oldKey);
 int extraBytes = 0; if((numBits % BITS_PER_BYTE) > 0){
 extraBytes = 1;
 }
 int totalBytes = (numBits / BITS_PER_BYTE) + extraBytes
 char *newStem = malloc(sizeof(char) * totalBytes);
 assert(newStem);
 for(unsigned int i = 0; i 
COMP20003 Code: 9773, Official Code Suburb: 20495, Official Name Suburb: Carlton, Year: 2021, Official Code State: 2,
South Melbourne 
With the following printed to stdout:
Carlton  1 records - comparisons: b64 n1 s1
South Melbourne  NOTFOUND
Note: the bit comparisons are 8 times the character comparisons here
Stage 4: Patricia Tree Spellchecker
In Stage 4, you will implement the another dictionary allowing the lookup of data by key ( Suburb 
Name ).
Your Makefile should produce an executable program called dict4 . This program should take three 
command line arguments: 
The ﬁrst argument will be the stage, for this part, the value will always be 4 (for Patricia tree 
lookup and comparison counting).
The second argument will be the name of the input CSV data ﬁle. 
The third argument will be the name of the output ﬁle.
Your dict4 program should:
Read the data in the same format as assignment 1 and stage 1, and save each entry in a Patricia 
tree using the Oﬃcial Name Suburb as the key.
The program should:Accept Official Name Suburb s from stdin , and search the tree for the matching key. 
Since your program should act as a simple spellchecker, an exact match may not be 
found. Follow the process "mismatch in key" as outlined below to implement 
spellchecking logic.
You must output all matching records to the output ﬁle, where a matching record may be 
an exact match, or the closest match determined by the mismatch key algorithm. If no 
matches for the query are found (i.e. the trie is empty), your program should output 
NOTFOUND . When outputting data records, you should follow the guidelines described in 
the slide Dataset and Assumptions. 
In addition to outputting the record(s) to the output ﬁle, the number of records found and 
the comparisons and node accesses when querying should be output to stdout .
Mismatch in Key
Since a Patricia tree is a type of preﬁx tree, any mismatch that occurs will contain the same preﬁx. For 
example, say we are searching for "trie", but our dictionary contains "tree". Here, a mismatch occurs 
at some binary bit in the third character. This mismatch will only occur once we are past the node in 
our tree that contains all preﬁxes with "tr". Therefore, when the mismatch occurs, we can get all 
descendants at the mismatched node and calculate the most likely candidate using the edit distance 
of the query and a key. This way, if we query "trie", we should return "tree". This is explained with the 
following graphic:
The patricia tree contains 4 strings as listed. If we query "trie", the mismatch happens on the bit 
highlighted in red. At this node, the only descendant data is "tree", and is therefore determined to be 
the closest match.If we query "Tree", the mismatch happens on the third bit. This means that all descendant data is 
taken to be possible candidates, so all 4 keys have their edit distance against the query calculated. 
"tree" is determined to be the closest match with an edit distance of 1, and is returned. 
Tips:
The results struct you created to hold comparisons can hold matching data
A node access count should be incremented each time a new node of the trie is looked at.
If two strings have an equal edit distance, return all results for the ﬁrst suburb name 
encountered with that edit distance (this will be the alphabetically earliest result).
Edit Distance
In Stage 4, you will need to calculate the edit distance of multiple strings to determine the closest 
match. You are welcome to use the code here to calculate the edit distance - you do not have to worry 
about its complexity:
int editDistance(char *str1, char *str2, int n, int m);
int int min(int a, int b, int c);
/* Returns min of 3 integers 
 reference: https://www.geeksforgeeks.org/edit-distance-in-c/ */
int min(int a, int b, int c) {
 if (a = 0  n >= 0  (str1 || m == 0)  (str2 || n == 0));
 // Declare a 2D array to store the dynamic programming
 // table
 int dp[n + 1][m + 1];
 // Initialize the dp table
 for (int i = 0; i 
COMP20003 Code: 9773, Official Code Suburb: 20495, Official Name Suburb: Carlton, Year: 2021, Official Code State: 2,
South Melbourne 
COMP20003 Code: 390, Official Code Suburb: 22313, Official Name Suburb: South Wharf, Year: 2021, Official Code State:
stdout:
Carlton  1 records - comparisons: b58 n4 s1
South Melbourne  1 records - comparisons: b52 n5 s1Stage 5: Complexity Report
The ﬁnal deliverable for this assignment is a small report analyzing the complexity of both 
dictionaries. You will run various test cases through your program to test its correctness and also to 
examine the number of key comparisons used when searching keys across diﬀerent dictionaries. You 
will report on the number of comparisons used by your linked list and patricia tree implementations 
for diﬀerent data ﬁle inputs and key preﬁxes. You will compare these results with what you expect 
based on theory (Big-O).
Your approach should be systematic, varying the size of the ﬁles you use and their characteristics (e.g. 
sorted/unsorted, duplicates, range of key space, length of preﬁx, etc.), and observing how the number 
of key comparisons varies given diﬀerent sizes of key preﬁxes. Looking at statistical measures about 
the results of your tests may help.
UNIX programs such as sort (in particular -R ), head , tail , cat and awk may be valuable. You 
may also ﬁnd small C programs (or additional "Stages") useful, the test data you produce and use 
does not have to be restricted to the dataset provided.
Additionally, though your C code should run on Ed, you may like to run oﬄine tests or other work 
outside of Ed and are welcome to do so.
You will write up your ﬁndings and submit them along with your Assignment Submission in the 
Assignment Submission Tab.
Make sure to click the "Mark" button after uploading your report. It is often the case for results to be released 
and more than a handful of students to ﬁnd an unexpected 0 mark for the report due to not technically 
submitting it for marking.
You are encouraged to present the information you've collected in appropriate tables and graphs. 
Select informative data, more is not always better.
In particular, you should present your ﬁndings clearly, in light of what you know about the data 
structures used in your programs and in light of their known computational complexity. You may ﬁnd 
that your results are what you expected, based on theory. Alternatively, you may ﬁnd your results do 
not agree with theory. In either case, you should state what you expected from the theory, and if 
there is a discrepancy you should suggest possible reasons. You might want to discuss space-time 
trade-oﬀs, if this is appropriate to your code and data.
It is recommended to look at extreme cases where each option will do better or worse - e.g. think 
about dense data where the Patricia tree will save many bit comparisons and character sparse data 
where string comparisons would terminate quickly in both data structures. It is also highly 
recommended to look at the theory covered so far and how you can match it - i.e. if a particular 
complexity is expected, that it appears at large input sizes and represents the growth behaviour - not 
just single point comparisons. Similarly, pay particular attention to the meaning of variables, 
character comparisons, string comparisons and bit comparisons may all have diﬀerent behaviours and certain investigations might expect diﬀerent relations on these.
You are not constrained to any particular structure in this report, but a well laid out plan with a strong 
structure tends to lead to the best results. A more general and helpful guiding structure is shown in 
the Additional Support slide, but you         
加QQ：99515681  WX：codinghelp
