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
$F(X,Y,Z)=(X\land Y)\lor (\neg X \land Z)$, $G(X,Y,Z)=(X\land Z)\oplus (Y\land \neg Z)$, $H(X,Y,Z)=X\oplus Y\oplus Z$, $I(X,Y,Z)=Y\oplus (X\lor \neg Z)$. For an cursory exploration of these functions, see Appendix A of the README.
5. 

## Appendix A: The F, G, H, and I Ternary Functions
For the ternary functions, one way of investigating their behaviours can be done by examining their truth tables. For example, this is a truth table for $F(X,Y,Z)$:
| X      | Y      | Z      | $X\land Y$ | $\neg X\land Z$ | $F(X,Y,Z)=(X\land Y)\lor (\neg X\land Z)$ |
| ------ | ------ | ------ | ---------- | --------------- | ----------------------------------------- |
| 0      | 0      | 0      | 0          | 0               | 0                                         |
| 1      | 0      | 0      | 0          | 0               | 0                                         |
| 0      | 1      | 0      | 0          | 0               | 0                                         |
| 0      | 0      | 1      | 0          | 1               | 1                                         |
| 1      | 1      | 0      | 1          | 0               | 1                                         |
| 1      | 0      | 1      | 0          | 0               | 0                                         |
| 0      | 1      | 1      | 0          | 1               | 1                                         |
| 1      | 1      | 1      | 1          | 0               | 1                                         |

Notice that $F(0,Y,Z) = Y$ and $F(1,Y,Z)=Z$. This shows that $F(X,Y,Z)$ is the conditional "if X then Y, else Z".  But look! Creating a logical table was entirely superfluous, because it's actually much easier to check $F(1,Y,Z)=(1\land Y)\lor (0\land Z)=Y$ and $F(0,Y,Z)=(0\land Y)\lor (1\land Z)=Z$. So a better way of investigating these functions could be to view them as a control flow within a program. \
 \
Notice that $G(X,Y,0)=(X\land 0)\lor (Y\land 1)=Y$ and $G(X,Y,1)=(X\land 1)\lor (Y\land 0)=X$ which is the same as the conditional if Z then X, else Y. \
 \
Next, we have $H(X,Y,Z)=X\oplus Y\oplus Z$. Since the XOR operator (denoted by $\oplus$) is symmetric, $H$ is also a totally symmetric function. So during our investigation, varying any of the variables is equivalent to varying any of the other variables. We find that $H(0,Y,Z)=0\oplus Y\oplus Z$. Recall that the XOR operator has the following logical 
| Q | R | Q\oplus R |
| ------------- | -------------- | -------------- |
| 0 | 0 | 0 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 1 | 0 |

For $Q=0$, we have that $Q\oplus R=R$. So generally, $0\oplus R=R$. So our expression for the function looks like $H(0,Y,Z)=Y\oplus Z$. Next, we check $H(1,Y,Z)=1\oplus Y\oplus Z$. From the logical table above, we can see that if $Q=1$, then $Q\oplus R=\neg R$. Thus, $H(1,Y,Z)=\neg (Y\oplus Z)$. If we think of this as a control flow, $X=0$ returns the result of the comparison $Y!= Z$ and $X=1$ returns the result of the comparison $Y==Z$. \
 \
Finally, we investigate the function $I(X,Y,Z)=Y\oplus(X\lor\neg Z)$. We take interest in the leading role that X takes in this function (my reasoning is that the \oplus part the function is more interesting to explore than the \lor part). We find $I(0,Y,Z)=Y\oplus(0\lor\neg Z)=Y\oplus\neg Z$. We can extend our previous table to see what the behaviour of this logical statement looks like 
| Q | R | $\neg R$ | $Q\oplus\neg R$ |
| ------------- | -------------- | -------------- | -- |
| 0 | 0 | 1 | 1 |
| 1 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 1 |

Really, all this is telling us is that we can pull the negation out from the expression $Y\oplus\neg Z$ to get $I(0,Y,Z)=\neg(Y\oplus Z)$. Next, we find $I(1,Y,Z)=Y\oplus (1 \lor\neg Z)=Y\oplus 1$. From the truth table for $Q\oplus R$, we can see that if $Q=1$, then $Q\oplus R=\neg R$. Since XOR is a symmetric operation, we then further simplify $I(1,Y,Z)=\neg Y$. Again, this can be interpreted as a control flow where $X=0$ returns the result of the comparison $Y==Z$ and $X=1$ returns the result of $\neg Y$. \
 \
While thinking of these functions as "control flow" gives a narrower scope on the abstract definition of the function, this has the positive effect that it is much more interpretable. 
