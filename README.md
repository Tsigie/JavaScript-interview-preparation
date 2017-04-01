# JavaScript-interview-preparation
My preparation for getting a full time job as a front end developer

* The null gotcha!

  null datatype qualifies as an Object.
  ```
  const bar = null;
  console.log(typeof bar === "object"); //returns true!
  ```

  If you want to reliably check if bar is an object, check for null condition first.
  ```
  console.log((bar !== null)&&(typeof bar === "object"));
  ```

  1. If bar is a function, the above code will return false.
  If you want to return true for bar being a function, simply add the following with an and condition
  ```
  (typeof bar === "function")
  ```
  
  2. Arrays are Objects too, so the expression will return true for arrays. 
  To return false for arrays, use the toString.class() method:
  ```
  (toString.call(bar) !== "[object Array]"))
  ```
 
* Since toString is defined in Object.prototype, whoever inherits Object, will by default get the toString method.

  But, Array objects, override the default toString method to print the array elements as comma separated string.
  [source](http://stackoverflow.com/questions/30010996/difference-between-object-prototype-tostring-callarrayobj-and-arrayobj-tostrin)
  
  toString() can be used with every object and allows you to get its class using the `call` function.
  ```
  var toString = Object.prototype.toString;

  toString.call(new Date);    // [object Date]
  toString.call(new String);  // [object String]
  toString.call(Math);        // [object Math]

  // Since JavaScript 1.8.5
  toString.call(undefined);   // [object Undefined]
  toString.call(null);        // [object Null]
  ```
  [source](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)

* `var a = b = 3;` is equivalent to 
  ```
  b = 3;
  var a = b;
  ```
  
  If the statement was enclosed within a function
  
  ```
  function funk(){var a = b = 3;}
  console.log(b);// We get 3
  console.log(a);//we get undefined
  ``` 
  
  b would be accessible and defined outside the function, but a would not be defined in the global scope because it has a var prefix in the declaration
  
  If the statement had const instead of var, neither of the variables would print outside the block's scope

* Coercion

  Type coercion means that when the operands of an operator are different types, one of them will be converted to an "equivalent" value of the other operand's type. For instance, if you do:

  ```
  boolean == integer
  ```
  the boolean operand will be converted to an integer: false becomes 0, true becomes 1. Then the two values are compared.

  However, if you use the non-converting comparison operator ===, no such conversion occurs. When the operands are of different types, this operator returns false, and only compares the values when they're of the same type.
  [source](http://stackoverflow.com/questions/19915688/what-exactly-is-type-coercion-in-javascript)
  
  Another example:
  
  ```
  [] + 5
  ```
  In JavaScript, + can mean add two numbers or concatenate two strings. In this case, we have neither two numbers nor two strings. We only have one number and an object. 
  Javascript knows how to concatenate strings, so it converts both [] and 5 into strings and the result is string value “5”.
  
* IIFEs are executed before other functions and have their own lexical scope which helps prevent the variables from leaking.
* IIFEs may get expressed in an expression before variables trickle down to their scope.

  ```
  var myObject = {
      animal: "bear",
      func: function() {
          (function() {
              console.log(this.animal, this);
          }());
      }
  };
  myObject.func();
  ```
  `this` inside of a `function` will always equal the global object (`window`) unless you change the context of `this` via `.bind()`, `.call()` or `.apply()`.
  
  `this` of a function inside a function always refers to the global object, even if the outer function is contained by an object.
  
  This function returns `undefined` for 'this.animal' and returns `Window` object for 'this'.
  Because animal is not present in the global scope and is limited to the object, it gets a value of undefined.
  
 * [A good read about 'this'](https://github.com/getify/You-Dont-Know-JS/tree/master/this%20%26%20object%20prototypes)

# All about `this`

`this` does not, in any way, refer to a function's lexical scope.

You cannot use a `this` reference to look something up in a lexical scope. It is not possible.

When a function is invoked, an activation record, otherwise known as an execution context, is created. This record contains information about where the function was called from (the call-stack), how the function was invoked, what parameters were passed, etc. One of the properties of this record is the `this` reference which will be used for the duration of that function's execution.

1. `this` inside a normal function always refers to Window
	```
	function func(){console.log(this)} // We get window

	function printWord(){return this.word}
	
	const word = “Hello”; //Global word
	const callerObject = {const word: “Goodbye”};

	printWord(); //Returns Hello
  

The scope of this in printWord is Window, so it accesses window.word which is “hello”


2. Using `apply()` we can explicitly set the `this` of func function to the object 
	passed as a parameter.
  
  ```
	func.apply(callerObject);// `this` becomes callerObject

	printWord.apply(callerObject); //returns “Goodbye”
  ```

3. When you use dot notation, `this` is what actually comes before the dot.
	```
	function func(){console.log(this)}
	callerObject.printWord = func;
	printWord(); // Returns callerObject as `this` 
	
.apply() has greater precedence over dot notation
	
  
	callerObject.printWord.apply(func);//'this' becomes func
	
	If you reference (copy) a function to a variable and run the variable, `this` inherits the scope from the variable and not the originally referred function
	
	func2 = callerObject.printWord;
	func2(); //returns “Hello” which is taken from the global scope and not from callerObject

	
4. `new` keyword.
  ```
function func3(){this.word = “Hi”;} //when func3 is instantiated, it also returns `this`. It returns the newly created object
const obj2 = new func3(); 
obj2 //returns object with {word: “Hi”}
  ```
When a function is used as a constructor (with the new keyword), its this is bound to the new object being constructed.

A function used as getter or setter has its this bound to the object from which the property is being set or gotten.

When a function is called as a method of an object, its this is set to the object the method is called on.

The this binding is only affected by the most immediate member reference. The most immediate reference is all that matters.

# Execution stack, variable environment & outer variable environment

* In an execution stack, an execution context for each function is created in the order in which they appear in the flow. However, when it comes to variable environments, each execution context maintains a link to the immediate outer (lexical) environment, where the function was written. This is the outer lexical environment, the environment in which the current function has been physically written. This outer environment is used when a variable inside our current execution context is not defined. So it is fetched from the lexically outer environment. If it doesn't find it there and there is an ever outer lexical environment, it checks for the variable there. This jumping of a variable value when it is not defined, from one environment to another, is called the **scope chain**. 

* Every execution context has its own variable environment. Every variable in JavaScript belongs to some variable environment. Global environment if it is not inside any execution context of functions. Where it is referenced from has an impact on its value. 

	```
	function b(){
	console.log(myVar);
	}
	function a(){
	var myVar = 2;
	console.log(myVar); 
	b();
	}
	var myVar = 1;
	console.log(myVar);
	a();
	```
	
	Here, myVar is globally set to 1. So it prints 1.
	Then it dives into a()
	a has myVar = 2, so it prints 2.
	Then it dives into b()
	b doesn't have a myVar, so it refers immediate outer lexical environment.
	There is nothing containing b(), so it refers to the global environment.
	Finally, it prints the global value of myVar, which is 1.

* Another example:

	```
	function b(){
	var myVar;
	console.log(myVar);
	}
	function a(){
	var myVar = 2;
	console.log(myVar); 
	}
	var myVar = 1;
	console.log(myVar);
	a();
	console.log(myVar);
	```
	First, it opens the global execution context and sets a variable environment for myVar
	myVar is set to 1.
	It prints 1.
	Then a() is called and its execution context is set, and a variable environment for myVar is created.
	myVar is set to 2.
	It prints 2.
	Then b() is called, its execution context is set up and a variable environment for myVar is created in b()
	myVar is set to undefined.
	It prints undefined.
	b() ends, so its execution context is popped. a() ends, its execution context is popped.
	Now we are in the global execution context, where myVar is 1.
	So it prints 1 again.

### How are events handled by JS? / How are asynchronous calls handled by JS

- The global execution context is set up.

- 'this' is created

- Variable environment for the global execution context is set up

- Any function call that appears first from top to bottom sets up its own execution context in the execution stack

- Its variable environment is set up

- The function 'a' is run from top to bottom

- Any other function that is called back by 'a' sets up its own execution context

- Variable environment is created and the function is run from top to bottom

- The function gives back control to 'a'

- When a finishes, it gives back control to the global execution context

- The global execution context finishes up all its statements

- Any event encountered by an eventListener in the code is sent out to the event queue

**Events**

- After the execution stack is empty, the engine starts processing events one by one from the event queue

- An event is read and any function call made by the event is taken in

- The function sets up its execution context in the stack, its variable environment is set up

- The function runs and similarly, any subsequent functions are run till the execution stack is empty

- The next event in the queue is taken in, and the process continues

### Precedence and assiciativity

	var a = 1, b = 3, c = 4;
	a = b = c;
	console.log(a,b,c);
	
Prints 4,4,4.
Because of associativity. '=' operator is right to left

- 3 < 2 < 4 returns true, because the '<' operator is left to right associative. It returns false (3 < 2) and then true (false < 4), leaving us with a 'true' result. Note that false is coerced to a value 0 which is less than 4 and hence yields true.

### By Value and by Reference

- Primitive types always get a copy - by value
	```
	var a = 3;
	var b = a; //b and a don't point to the same 3. b gets its own 3 stored in a different memory location
	```
- Objects always get reference to same object - by reference
	```
	var cars = {ford:"Ford GT", nissan:"GTR"};
	var raceCars = cars; //both point to the same object
	raceCars.hennessey = "Venom GT";
	cars; 
	//returns
	//{ford: "Ford GT", nissan: "GTR", hennessey: "Venom GT"}
	//Awesome! Isn't it?
	```
- = operator creates new memory space. It is a special case where by reference does not apply.
	```
	cars = {dodge: "Viper", mazda:"RX-7", toyota:"Supra"};
	raceCars;
	//returns
	//{ford: "Ford GT", nissan: "GTR", hennessey: "Venom GT"}
	```
	
## Closures

JS preserves the variable environment of an execution context even after that context has finished executing and is popped off the stack.

These retained variables are available to functions that are authorized to access them.

	function build(){
	var arr = [];
	for(var i=0; i<3; i++){
	arr.push(function(){console.log(i)});
	}
	return arr;
	}

	var f = build();

	f[0]();
	f[1]();
	f[2]();

- build function pushes the definition of 3 functions into array arr. 

- Note that it does not console.log the value of i. i is just a literal string passed as function definition.

- Then arr is returned, gets saved in f.

- As build finishes executing, the value of i increments to 3, and all function definitions have been returned.

- build() is cleared from the execution stack.

- f has all the 3 functions.

	```
	f[0]();
	f[1]();
	f[2]();
	```
	
- Each of these functions called needs to access the value of i.

- But execution stack of build has finished executing and is no longer there.

- Yet, the final value of i is retained in the global scope. 

- The variable environment of build is preserved due to closure.

- Thus, all 3 functions print 3, which was the final value of i.

- If you wanted to print 1,2,3, use let i. let datatype segments the memory.
It stores each new value of i in the for loop in a different memory location.

- IIFE is another way to have 1,2,3 printed. Just enclose the return function statement inside an IIFE
Pass the counter to the IIFE and refer to that variable inside the actual function definition to be returned.
For each iteration of the loop, a new execution context for each IIFE will be created, thus referencing the correct values of the counter.

This is very similar to getting closure after watching Star Wars series.
The episodes you are yet to watch, are functions authorized to access the part of your neural network.
As the story progresses, the information in your brain is updated.
When you finish watching episode 3, you get closure. 
You are no longer desperate or curious.
The first series is gone, has finished, and yet you retain the information, the latest version of it.
And this information flows through the next trilogy even after the first one is over.
The first trilogy is gone, yet the second one can access it. 
Because the information is retained in your brain.
