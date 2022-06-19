# Certora guide:

learning the process  of formal verifying properties of defi protocols on certora 

## index :
- [ ] Setup. 



## 1. installation: 


- for practicing the syntax for their proof specific language , their [demo]()  IDE  is sufficient. 
- or getting the key  of accessing the certora prover API from the core team  for demo(by joining the [certora discord]() . 


- cloning  the necessary files from the examples repo that will be refered once the basic syntax and the concept of writing and validating proofs are verified. 

## 2. understanding the CVL (certora verification language): 

basic structure of defining the rule 


```javascript

rule function_ex() {
env e; /**this defines the EVM params passed by the called (msg.sender , msg.value , ****) */

// now we will need some output from the given function in order to ascertain the functionality . 
<<data type>> param = functionName(e);

assert param == 'value', "the function didnt worked as expected"; 
}
```


- for more generic definition , we have the following parametric rules :

```javascript
rule othersChangingFuncState() {
method f;
env e;  // the execution environment
address other;

require e.msg.sender != other;



uint256 initValue = getFunc(other);

calldataarg arg; 

func(e,arg); // execution for changing the function state

uint256 afterValue = getFunc(other);

assert initValue <= afterValue, "Reduced the balance of another address";
}
```

- **defining the invariants** : this will be the properties that will be always true once the operation is concluded. 

defined as:


```javascript

rule invariantAsRule(method f) {

require exp;
calldata arg;
f(e,arg);
assert exp;


}
```

## 2.1 now defining the building blocks of the function:

there is another block defined in the spec files (called as  methods) in order to define the function interfaces 

```javascript
func name(params) returns uint256 (envfree -> optional only  when if the msg.* call params are not needed).

```

## 2.3. understand the implementation  of specifications for   mapping datastructure: 


-  defining unit test. 
```javascript

rule checkAdd(uint key, uint value) {
env e;
insert(e, key,value);
assert get(key) == value, "value of key not equal to the inserted value";
assert contains(key), "key is not contained after successful insertion";

}  


// now checking the revert conditions (for the functions that are internal and dont reutrn anything).
// using insert functionality.

rule  insertRevertConditions(uint key, uint value) {

    env e;
    insert@withRevert(e,key,value);
    bool success = !lastReverted; 
    // important to note that given the function is not payable (in case of the transfer) , thus you can transfer the amount by the user
    assert (value != 0  && e.msg.value == 0  && !contains(key)) => succeeded;

}

// then comes the proving rules consisting of the reversal of previously verfied rule. 

// thus for the inverse, taking the example :

rule inverse(uint key , uint value) {
    env e;
    insert(e,key,value);
    env e2; // using different env variables in order to take more general condition (multiple R/W).
    require e2.msg.value == 0;
    remove@withRevert(e2,key);
    bool removeSuccedded, "remove after insert must work";
    assert removeSucceeded, "remove after insert must succeeded";
    assert get(key) != value, "value of removed key must not be the inserted value";
}

```
Also prover does not iudeally go in fail state in case there is infinitely unrolling of the computation . Certora prover reverts with the specific value on which the first time users finds the error. 


- generally we can detect easily the issues about the key and corresponding values edge cases and accordingly avoid those edge cases based on the condition , by doing some minor changes . 

```javascript
rule inverse(uint key, uint value) {
    uint max_uint = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
    require numOfKeys() < max_uint;
    env e;
    insert(e, key, value);
    env e2;
    require e2.msg.value == 0;
    remove@withrevert(e2, key);
    bool removeSucceeded = !lastReverted;
    assert removeSucceeded, "remove after insert must succeed";
    assert get(key) != value, "value of removed key must not be the inserted value";

```


defining the invariant is done by the first order logic specifications (in maths set theory). 

and then defining the invariants : 

```javascript
invariant mappingDiffInArr(uint x) {
getVal(x) != 0 <=> (exists uint i. 0 <= i && i < getNumOfKeys() && keys(i) == x)

}

```

but issues comes is the incompatiblity of the invariants functions to interact with the underlying specifications of the compiled code of the solidity defined contract . so we need to implement the mock represnetation of the actual mapping / function logic . 

for eg : 

ghost mapping(uint => uint) _map;

and then implementing the invariant function for ghost functions :

```javascript
invariant checkMapGhost(uint someKey) get(someKey) == _map[someKey]
```

but in order to satisfy the specs for the state mutating functions (where the linking of the ghost implementation needs to be similar to the solidity counterpart) . we need to wrap the implementation using `Sstore`:

```javascript

hook Sstore map[KEY uint k] uint v SOTRAGE { 
    _map[k] = v;
}

hook Sstore map[KEY uint k] uint v STORAGE {
    _map[k] = v;
}

/// and defining further ghosts for  getting mapping of array and storage for elements of array 
```
hook Sload uint n keys[INDEX uint index] STORAGE {
    require array[index] == n;
}

hook Sstore keys[INDEX uint index] uint n STORAGE {
    array[index] = n;
}



## 3. Setting up the project layout  and FV implementation thinking : 
- [ref](https://github.com/Certora/Tutorials/tree/master/06.Lesson_ThinkingProperties). 

thinking about the properties: 
- expressing each system requirement into the preferred representation
    - either plain english with points and call graphs analysis (using surya with verbose specs). 
        - defining the properties that are factually correct to be tested in the given codebase is most important rather than just implementing the tests quickly. 
            - the property coverage has to be robust for the given rule defined. 

some links and steps to be followed:

 Go through the presentation [Categorizing Properties](https://github.com/Certora/Tutorials/blob/master/06.Lesson_ThinkingProperties/Categorizing_Properties.pdf) to learn about different classes of properties that can be found in real-life systems.

 Follow the instructions in an example project [AuctionDemonstration](https://github.com/Certora/Tutorials/blob/master/06.Lesson_ThinkingProperties/AuctionDemonstration) to have a guided exercise with a demonstration of thinking about properties.

 Follow the instructions on [Thinking Properties Exercise](https://github.com/Certora/Tutorials/blob/master/06.Lesson_ThinkingProperties/ThinkingPropertiesExercise) to exercise some more on coming up with properties, categorizing them, and prioritizing them.



### Milestone to be implemented.
 - [ ]Create a .spec file for some example projects[1](https://github.com/Certora/Tutorials/blob/master/06.Lesson_ThinkingProperties/ThinkingPropertiesExercise/MeetingScheduler)[2](https://github.com/Certora/Tutorials/blob/master/06.Lesson_ThinkingProperties/ThinkingPropertiesExercise/TicketDepot), and proving the properties to get the solutions reviewed from the certora team.


### defining whole specifications : 
- [ ] version pragma 


### learning general grammer of the writing CVL programs(WIP) : 

[Links](https://docs.certora.com/en/latest/docs/ref-manual/cvl/overview.html).
- defining the version of sol
- import statements for the other CVL files . 

### TODO: deifining the specific files in the given operation:  

## best practices : 
- defining the mathematical representation of the function proofs : 
    - [using first order logic](https://formal.kastel.kit.edu/beckert/teaching/Formale-Verifikation-SS09/03FirstOrderLogic.pdf), these are the techniques for determining the 

**More concise tutorial series about FV testing** (https://github.com/Certora/Tutorials)