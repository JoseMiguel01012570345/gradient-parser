# Gradient Parser

In this file you will see the characteritics of this new parser proposal. To organize this trip we will step over the next topics:

- `Shift | Reduce criteria`
- `Matrix operator Precedence`
- `Automaton Gradient`
- `Grammar style and its expressiveness`
- `Conflicts`
- `Advantages and disadvantages`
- `Implementation`

## Preview

When a string is given there are certain part that we need to check before others to determine if such string belongs to the language , so we need to give a precedence order to thense parts of the string and analize it. Let's say we have a vocabulary $V$ where reminds all of the symbols we have in our language, which is composed of terminals. 

The proposal of this articule is to define the language as $<Gr,G>$ pair where $Gr$ is the grammar productions of the language and $G =<V,P>$ is a graph relationships between terminal where  $V$ are all the terminals ( vertices of the graph ) and $P$ is the relationship between two terminals. If $<t_0,t_1> \in P$ then the parser can reduce stack when it has $t_0$ in stack, sees $t_1$ and has a production that matches the top of the stack containing $t_0$. If $<t_0,t_1> \notin P$ then the parser shifts the character. So the problem to determine if a given string belongs to our language is to determine if the graph builded from the given string is a sub graph of the graph builded from the grammar defined. 

The Gradient Parser is a buttom-up parser that takes some features of others shift | reduce parsers that peformes reduction or shift based on the criteria of symbols precedence indicated in the `matrix precedence` (adjacency matrix) and also takes similar features from others parser such as the look-ahead feature and the automaton is quite similar but with some small changes that makes it different. This all allows this parser to retrives more expressiveness of the grammar and such time complexity is $O(n)$ where $n$ is the length of the code. Its components are: a stack whose contains seen terminals and non-terminals . It has a `stack pointer` that stores the position of seen terminals in the string in the form $( index , t )$.

## Shift | Reduce criteria

The criteria for peforming a shift or reduce action is given as in the definition 1.1:

### Definition 1.1

Let the parser be in the state $\sigma|\beta$ or $\sigma\alpha|\beta$ where $\alpha \in N$ and $\sigma,\beta \in T$ then a shift action is peformed if and only if $\sigma$ has lower precedence than $\beta$, and a reduce action is made if and only if $\sigma$ has a bigger or equal precedence than $\beta$

Note 1.0: Under this shift | reduce criteria it is not true that we will ever reduce every time the parser determines reduce action, because what is modeled here is the chances to reduce. If under a reduction we haven't defined any preduction that matches with the top of the stack, we won't change the stack. If this statement is not obvious yet , it will become clear in the lines to come

Note 1.1: The reazon way the criteria to peform a reduce action is considered to be equal precedence is because we need the parser to associate to the right , otherwise strings of the form $int - int + int\$$ , where $\$$ indicates end of the string and so will always reduce , will be reduced first at the sum and then at minus operator, which is wrong .

## Matrix Procedence definition

The matrix precedence is $|T| * |T|$ dimentional matrix where $\alpha_{\sigma \beta} = 1$ if and only if $\beta$  has a bigger or equal precedence then $\sigma$  $;$ in other case $\alpha_{\sigma \beta} = 0$ if and only if $\beta$ has a lower precedence then $\sigma$ , where $\alpha \in N$ with  $\sigma,\beta \in T$ and $\sigma$ is pivote and $\beta$ is pointer.

## Automaton Gradient

A state is of the form $(\sigma|\beta w , pivote , pointer )$ where $\sigma,\beta \in T \cup N$ and $\sigma$ is the pointer if and only if $\sigma \in T$ and $\beta$ is the pivote if and only if $\beta \in T$. If $\sigma \in T$ then stack pointer contains $\sigma$ and its position in stack. 

The transition functions is $GOTO( s )$ where $s$ is posible states

Step 1: Add special symbols
- Adds two special symbols that allows walking thougth the string. Let $s$ be the string then convert to # $ S $ where $ always peforms `shift` action if and only if a pivote is not $ , in such case it preforms `reduce` action , otherwise # always peforms `shift` action , the last $\$$ symbol peforms reduce action whenever # is not a pointer.

Step 2: Find out action

-  If $\sigma|\beta w$ or $\sigma\alpha|\beta w$  are possible states where $\sigma,\beta \in T \cup N$ , $\alpha \in N$ and $\sigma$ is pointer and $\beta$ is pivote , then we peform an action if $\sigma,\beta \in T$ , accordingly to the matrix precedence.

Step 3: Peform action

The next step is divided into two posible cases:

   1. If `shift` action, then $GOTO(\sigma\beta|w, \pi \in T,\beta)$ or $GOTO(\sigma\alpha\beta|w , \pi \in T,\beta)$ , ( where $\pi$ is the next terminal rigth after $\beta$ ) whether the case.
 
   2. a ) If reduce action then chunk the stack from the pointer position , let's say $\sigma$.position $-$ look_ahead $+$ $i$ , where $i$ iterates through the stack pointer to the top ,and verify for posible reduction of the form $X -> \alpha$ .
        
        b ) If a reduction is made we remove  from stack all of the terminals that reminds whether the pointer position to the top of the stack and the new pointer is the top of the stack pointer , let it be $\epsilon$ , then do $GOTO(\sigma X|\beta , \beta ,\epsilon )$ and step over step 2. 
   
        c ) If no reduction was made then do $GOTO(\sigma \alpha|\beta , \pi , \beta)$ , where $\pi \in T$ and it's position is $\pi$.position = $\sigma$.position $-$ look_ahead $+$ $i_k$ and step over step 2.

        d) If we reach the top of stack pointer, we move the top of the stack pointer to the position ($\sigma$.position $-$ look_ahead) and step over step 2

Step 4: check final state

- If we reach the state # $ S $ |, $\epsilon$ , { #, $ } , where $\epsilon$ indicates no string  , it means  we have finished and the string belongs to the language

Can be guarateed that we won't iterate forever because # symbol helps to walk forward in the string in case we reach such symbol while trying to reduce.

The steps are resumed here:

 1. Add sprecial symbols
 2. Find out action
 3. Peform action
    - a) iterate from pointer to the top of the stack pointer doing :  pointer.position $-$ look_ahead $+i$ , where $i \in \N \cup 0 , i = i_0=0,i_1=1,...,$look_ahead
    - if reduction , set $i = i_0$
    - if no reduction , and $i=i_k$  then do $i=i_{k+1}$
    - if reach top of stack pointer , set the top of the stack pointer to: pointer.position $-$ look_ahead $+i_0$

 4. Check final state

## Grammar Expressiveness

Using this parser allows grammar of the style $T \oplus T$ as we will see in the implementation bellow as well as an important feature to desambiguate grammar using the idea of look-ahead. This last feature is taken in a sligly different way , when the parser suggest reduce, there is limit of terminal that a pivot can reduce before finding a pointer that makes him shifts , length from our shift-pointer to the current pivote in stack is as long as the parser will re-observe for reduction . In more ambiguos grammar it can be peformed a much more look-ahead sight , up to the length the parser will re-observe , because it has no sense the parser looks longer then the terminal pointer that makes it peforms a shift action , due to there is no concept here of viable prefix has in others shift | reduce parsers 

## Conflicts

- The grammar has no shift-reduce conficts , the action to peforms is not in productions, but in matrix precedence which deals with it . 
- If any two production is sufix of the other production then we have to implement a look-ahead as long as to cover the production that do contains the prefix of the other production .
- If there exits two productions like: $X ->\alpha$ and $Y ->\alpha$ then we have a permante conflict in the grammar and the parser choses only one, which might lead to an unexpected result , so the grammar is not `gradient`

## Advantages

   - The new proposal is a more declarative way to define a language because productions can have much more symbols of the language , and be better understood and so easer to define.
   - The graph is easy to define , you only need to declare the edges are or what the edges are not in ( min($e \in P$,$e \notin P$) ).
  
## Disadvantage

   - If there are two equal productions and different derivation , the automaton crashes. However we can insert the idea of what we might expect to read and solve the problem

