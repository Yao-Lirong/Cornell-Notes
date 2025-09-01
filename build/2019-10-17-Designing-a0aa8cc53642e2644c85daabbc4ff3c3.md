---
title: Designing and documenting interfaces and implementations
date: 2019-10-17
tags: 
- Cornell
- 19FA
- CS2112
---

From Lecture: [Designing and documenting interfaces and implementations](https://www.cs.cornell.edu/courses/cs2112/2019fa/lectures/lecture.html?id=intf_design)

---

## An Example of Writing Interface

### 1. Overview: what concerns? 

   ```java
   /** An n*n mutable 2048 puzzle. */
   class Puzzle{}
   ```

<!--more-->

### 2. Choose operations

   - creators create objects
     - constructors
     - factory methods (usually static, loose coupling; don't expose the choice of which class is being constructed)

   ```java
   	public Puzzle(int n){}
       public static Puzzle create(int n){}
   ```

   - observers(queries):
     - primary purpose: report the state of object
     - no side effect (doesn't change anything)
   - mutators (command): 
     - primary purpose: side effect on the state of the object
     - only makes sense when the object is mutable 
     - should maintain class invariants 

   **Command-Query Separation**: You have to decide whether this class belongs to command or query, don't write both of these things in a single method 

   **Problem with getters**: 

   representation exposure in Mutable Objects (sometimes you directly return the reference to this object and now doing modification to the returned object and unintendedly change the original object)

### 3. Write Specifications:

   - Returns/Creates: postcondition
   - Requires/Checks: precondition
   - Effects/Modifies: side effects
   - Examples: (as needed)
   - Exceptions: 
      	- in return clause: when there is a client error (not programmer error)
            	- in checks clause: when precondition violated 

   ```java
   /** Returns: the puzzle size n*/
   int size();
   
   /** Effects: adds a random tile to the board*/
   void addRandomTile();
   
   /** Effects: adds a random tile to the board
   	Returns: true if there was room*/
   boolean addRandomTile();
   
   /** Effects: adds a random tile to the board
   	Throws: BoardFull if there is no room*/
   void addRandomTile() throws BoardFull;
   
   /** Effects: shifts the tiles in direction d*/
   void shiftTile(Direction d);
   ```

---



### 4. Documenting Impls:

**Audience**: maintainers, not clients

**Goals**: keep impl details out of the spec (abstraction barrier)

1. `Represents`: an abstraction function: concrete fields -> client view

   ```java
   /** A rational number */
   class Rational {
       int num,dem;
       //Represents: the rational number num/dem
   }
   ```

2. Class Invariant: 

3. Spec for private/protected methods 

4. Algorithms Explanations: (If sometimes the spec is not enough to understand the method(specific algorithm), you should write algorithm explanations inside the method(This would be something client doesn't have to know but maintainers may want to know so you don't write them in the spec))

   Write paragraphs instead of interleave comments 
   
   ```java
   /** revtrieves the greatest common demoninator of x and y*/
   int gcd(x,y){
       /**This method uses Euclid's to find the ...    */
   }
   ```