---
layout: post
title: Algorithms notes
date: 2019-12-16
description: 
---
**Disclaimer:** current page is not for educational purposes, it has a intention of notes for myself about algorithms.

## Iteration over strings (Перебор строк)
**Lexicographical** order: (abc < acb)  
Strict definition:  
```
s < t  
s[i] = t[i], i=0,1..k-1, s[k]<t[k]
```  
For three chars 'a', 'b', 'c' the code to produce all combinations in 
lexicographical order is:  
```java
for (char i = 'a'; i <= 'c'; i++) {
    for (char j = 'a'; j <= 'c'; j++) {
        for (char k = 'a'; k <= 'c'; k++) {
            System.out.println(i+""+j+k);
        }
    }
}
```  
```java  
aaa
aab
aac
baa
...
ccc
```
In more general case, when it is required to iterate over all strings with length **n** which consist of first **m** chars of alphabet.    
The task can be reformulated differently: Output all sequences with a length on **n** which consist of numbers from 0 to **m**.
```java
private static Integer[] array;
private static Integer m = 3; // max integer in the sequence
private static Integer n = 3; // length of the sequence

public static void main(String[] args) {
    array = new Integer[m];
    rec(0);
}

private static void rec(int idx) {
    if (idx == n) {
        Arrays.stream(array).forEach(System.out::print);
        System.out.println();
        return;
    }
    for (int i = 1; i <= m; i++) {
        array[idx]=i;
        rec(idx+1);
    }
}
```
```java  
111
112
113
121
...
333
```
## Permutation
All sequences of integers from 1 to n, where each integer is used only once. Such sequence is called - **permutation**.  
```java
public class IntegerPermutation {
	static int size = 3;
	static Integer[] array = new Integer[size];
	static Boolean[] used = new Boolean[size+1];
	
	public static void main(String[] args) {
		initUsed();
		rec(0);
	}
	
	private static void rec(int idx) {
		if (idx == size) {
			Arrays.stream(array).forEach(System.out::print);
			System.out.println();
		}
		for (int i = 1; i<=size; i++) {
			if (used[i] == true) {
				continue;
			}
			used[i] = true;
			array[idx] = i;
			rec(idx+1);
			used[i] = false;
		}
	}
	
	private static void initUsed() {
		for (int i=0; i<size+1; i++){
			used[i]=false;
		}
	}
}
```   
Array `used` is needed to restrict having permutation with same integers. Output is:
```java
123
132
213
231
312
321
```
### Correct brace sequence
Lets consider a correct brace sequence from Math point of view. E.g. `((()))`, `()()()`, `(())()`. One of implementations:  
```java
boolean isCorrect(String s) {
    int balance = 0;
    for (int i=0; i<s.length(); i++) {
        if (s.charAt(i) == '(') {
            balance++;
        } else {
            balance--;
        }
        if (balance<0){
            return false;
        }
    }
    return (balance == 0);
}
```  
Also there is a well-known implementation of this algorithm with a stack.