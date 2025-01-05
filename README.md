# my-md5
## Introduction
This is an implementation of the MD5 hashing algorithm detailed in the document [RFC 1321: The MD5 Message-Digest Algorithm](https://www.rfc-editor.org/rfc/rfc1321). While the instructions here are for 32-bit systems, the majority of computers now use a 64-bit architecture. This changes some of the details, but the general outline of the algorithm remains the same. It is the purpose of this document that it details my exploration of the MD5 algorithm, its history, and its vulnerabilities. \
 \
As the document RFC 1321 explains, the MD5 hashing algorithm is a takes a variable length message as an input, and produces a 128-bit value as an output. I would like to note here that MD5 is no longer widely used, as it has known collision vulnerabilities that significantly impact its security. I will also be outlining these vulnerabilities at the end.
## The Algorithm
Here, I will outline the procedure for creating an MD5 hash out of a message, which is detailed in Section 3 of RC 1321. 
1. We want to message to be congruent to 448 bits (mod 512). If the message is not this value, then append a padding to the message to make it congruent to 448 bits (mod 512). The padding is 1 "1" followed by as many "0"s required to reach that condition. 
2. Append 64 bits to the end of the padded message, which is equal to the full length of the original message (before padding). If that length is larger than $2^{64}-1$, then truncate the number so that only the least significant 64 bits are used. The message length at this step should now be equal to a multiple of 512 bits.
3. Set the initial states $A=0x01234567$, $B=0x89abcdef$, $C=0xfedcba98$, and $D=0x76543210$. Recall that the "0x" prefix denotes a number written in a hexadecimal representation. The actual size of the state space is 32 bits. 
4. Define the four ternary functions that take a word (a 32 bit number) and return another word. 
$F(X,Y,Z)=(X\land Y)\lor (\neg X \land Z)$, $G(X,Y,Z)=(X\land Z)\oplus (Y\land \neg Z)$, $H(X,Y,Z)=X\oplus Y\oplus Z$, $I(X,Y,Z)=Y\oplus (X\lor \neg Z)$.
5. 

## Notes 
For the ternary functions, here are some logic tables describing their behaviour. The behaviour of $F(X,Y,Z)$ looks like
| X      | Y      | Z      | $X\land Y$ | $\neg X\land Z$ | $F(X,Y,Z)$ |
| ------ | ------ | ------ | ---------- | --------------- | ---------- |
| 0      | 0      | 0      | 0          | 0               | 0          |
| 1      | 0      | 0      | 0          | 0               | 0          |
| 0      | 1      | 0      | 0          | 0               | 0          |
| 0      | 0      | 1      | 0          | 1               | 1          |
| 1      | 1      | 0      | 1          | 0               | 1          |
| 1      | 0      | 1      | 0          | 0               | 0          |
| 0      | 1      | 1      | 0          | 1               | 1          |
| 1      | 1      | 1      | 1          | 0               | 1          |
