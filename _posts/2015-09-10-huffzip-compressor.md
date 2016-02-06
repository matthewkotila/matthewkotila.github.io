---
layout: post
title: HuffZip Compressor
description: Compress files using the Huffman Compression algorithm.

---

HuffZip is a Mac OS X application for compressing (primarily) text files using David A. Huffman's [coding algorithm](https://en.wikipedia.org/wiki/Huffman_coding). This application can compress any reasonably-sized file, and save it for future decompression.

## History
The Huffman coding algorithm is one of the earlier forms of compression taught in standard data structures college courses. The idea behind the algorithm is to rewrite the binary definitions for each unique ascii character for a given file. The American Standard Code for Information Interchange (ascii) is a set of 256 integer-character mappings, each representing a character that can be found in a standard text file on a computer. Each character is defined by a unique sequence of eight bits—<code>00000000</code> all the way to <code>11111111</code>. This provides for 256 possible combinations (2⁸). Say, for instance, you are trying to compress a file that contains the entire works of William Shakespeare. Based on Cornell University's [letter frequency data](http://www.math.cornell.edu/~mec/2003-2004/cryptography/subs/frequencies.html), letters such as e, t, a, o, and i should show up far more frequently than z, j, q, or x. Because of this probabilistic inherence in language, text can be compressed by shortening the 8-bit definition of the most frequent characters.

## Example
In an example text file, there 100 'a' characters, and 1 'b' character. The ascii value for 'a' is defined as the integer 97, or written in binary: <code>01100100</code> (98 and <code>01100010</code> respectively for 'b'). Currently, the text file would be 808 bits or 101 bytes in size. If the binary definition for the character 'a' were rewritten to be a single 0 and the binary definition for 'b' were rewritten to be a single 1, the entire content of the file could be reduced to just a little over 12 bytes. That's an impressive reduction on an earlier file that was over eight times larger than the compressed version. Of course, for portability reasons, each newly redefined ascii value must have its new binary definition stored somewhere in the compressed file for later decompression.

## In Action
HuffZip has two main components: encoding and decoding. When the application is run by opening it regularly, a file selection window appears:

![placeholder](/img/HuffZipMainScreen.png "Main screen")

You can select a (reasonably sized) file from the computer and the application creates a new file with the ".hzip" extension containing the compressed contents of the original unaltered file. If you open a file with the ".hzip" extension, HuffZip automatically launches and starts decompressing the file with which it was opened, and saves the decompressed file to the compressed file's directory.

## Everything In Between: Encoding
This application was written entirely in Objective-C and C and works on all machines able to run Mac OS X 10.10. The uncompressed file contents are read into RAM using good ole <code>fread</code>. I tend to find uncomplicated C code to be faster than most other frameworks' or languages' attempts at similar tasks. Each unique character from the uncompressed file is given a tally mark each time it is read, and based on these frequencies, a binary heap is created. Because of the O(n) time complexity of building a heap, <code>heapify</code> is called once instead of calling <code>insert</code> on each ascii to assemble the heap.

The actual Huffman binary tree (composed of node structs containing ascii value, frequency, and left and right pointers) is constructed by taking the two most frequent ascii values from the heap, and assigning their parent a newly created root node with the frequency of both of its children added together in a loop:

{% highlight objc %}
// loop for creating Huffman binary tree
while (heap->size > 1) {
  root = (heapNode*)<code>malloc</code>(sizeof(heapNode)); // create new parent
  destructorList[destructorIndex++] = root; // memory management

  // set children of new parent node
  [heap getAndDeleteMin:&root->left];
  [heap getAndDeleteMin:&root->right];
  root->freq = root->left->freq + root->right->freq;

  [heap insert:root]; // insert parent back into tree
}
{% endhighlight %}

Then, the recursive getEncodings function is called to traverse the tree for each unique path to a leaf node (only leaf nodes contain ascii values). Each unique binary code retreieved from a tree traversal is saved to an <code>unsigned int</code> by using the \|= operator. This saves space and is extremely quick for writing. After all of the unique binary codes for ascii values have been retrieved from the tree, they are saved to the beginning of the <code>unsigned char</code> data stream for later saving. Specifically, the ascii value, length of newly defined binary code, and binary code are saved for each unique ascii that appears in the uncompressed file. 1 byte for the original ascii, 1 byte for the code length, and 4 bytes for the code (I allow up to 32 bits for a newly defined binary code). I call this the "dictionary" stored at the front of the file.

The next step in compressing the file is to read the contents, byte by byte, and write the new definitions for each ascii to the compressed data stream as it is encountered. This is done using an <code>unsigned long</code> buffer. For each byte in the uncompressed file, the <code>unsigned long</code> gets the new definition for that uncompressed character/byte stamped onto it using the bitwise \|= operation. After at least eight bits are stamped onto the buffer, the most significant byte of the <code>unsigned long</code> is written to the compressed data stream, and the buffer is bitshifted eight bits to start all over again:

{% highlight objc %}
// loop for encoding each byte of file
for (int i = 0; i < fileLength; i++) {

  // stamp binary code onto buffer
  buffer |= (codes[fileContents[i]] << bufferLength);
  bufferLength += codeLengths[fileContents[i]];

  // write each byte of buffer to data stream
  while (bufferLength >= 8) {
    encodedContents[encodedLength++] = buffer;
    buffer >>= 8;
    bufferLength -= 8;
  }
}
{% endhighlight %}

Once every byte in the uncompressed file has been read and had its new definition saved to the compressed data stream, the application saves the compressed data stream (a <code>malloc</code>'d <code>unsigned char *</code>) to the disk using <code>fread</code>. After allocated memory has been freed in the destructor method, the compression component of the application is completed.

## Everything In Between: Decoding
Similarly to the compression component of HuffZip, decompression begins by reading the contents of the compressed file into RAM. The application starts off by parsing the dictionary stored at the beginning of the compressed file. Because each ascii, its code length, and (up to 32 bit) binary code are stored in the dictionary, a trie is made on the spot based on the sequence of zeros and ones for each unique ascii definition. Essentially, a root node is created, and depending on the boolean value of the bit of a given ascii binary code, a left or right child is created for the root. This process continues, creating a new child when one isn't present. At the end, a reconstructed binary Huffman tree is created, ready to be traversed during the decoding.

The application then reads the compressed file bit by bit, and traverses down the Huffman tree after each bit is read—going right if the currently encountered bit is <code>1</code>, or left if it is <code>0</code>. After each descension, a check is made to see if the current node is a leaf node containing an ascii value. If it does, a regular ascii value is written to the decompressed data stream (a <code>malloc</code>'d <code>unsigned char *</code>). Eventually, the end of the file is met, and all encoded contents have been decoded and written to the data stream. The application writes the decompressed data stream to the disk using <code>fwrite</code>, and frees all memory allocated during the execution of the application.

## Tangibles
You can view the code on [GitHub](https://github.com/matthewkotila/HuffZip) and download the .app [here](/public/HuffZip.zip).

<hr>
