## Background ##

There are various solutions to creating Javascript 'classes' with public and private variables and methods. The current common solutions are:

 - putting public methods into 'this' and using closures with the 'that=this' hack for private methods.
(see: [http://javascript.crockford.com/private.html](http://javascript.crockford.com/private.html))

 - Private data via ES6 class constructor
 (see: [http://exploringjs.com/es6/ch_classes.html](http://exploringjs.com/es6/ch_classes.html))
 
 - Private data via a '_' naming convention
  (see: [http://exploringjs.com/es6/ch_classes.html](http://exploringjs.com/es6/ch_classes.html))

 - Private data via WeakMaps
  (see: [http://exploringjs.com/es6/ch_classes.html](http://exploringjs.com/es6/ch_classes.html))

 - Private data via Symbols
  (see: [http://exploringjs.com/es6/ch_classes.html](http://exploringjs.com/es6/ch_classes.html))

<br />
I ended up dissatisfied with all of these solutions for the following reasons:

 - Ugly syntax - ending up with lot's of 'this._', defining methods inside the constructor etc
 
 - Error prone - especially the issue of 'this' inside private methods not referring to the instance unless you remember the confusing 'that=this' hack

 - Complicated - the use of WeakMaps or Symbols make the code hard to read with lot's of extra 'boiler' code

 - No clear separation of the public interface (proxy) of the class from the implementation

Solution
---------
Below is a better, simpler solution with the following advantages:

 - no need for 'this._', that/self, weakmaps, symbols etc. Clear and straightforward 'class' code 

 - private variables and methods are _really_ private and have the correct 'this' binding

 - No use of 'this' at all which means clear code that is much less error prone (as a side note, before ES6 'this' was a necessary evil, now it is simply evil, an evil we can do without).
 
 - public interface is clear and separated from the implementation as a proxy to private methods

## Code ##

**Boilerplate - Define 'New' for all functions:**

    if (typeof Function.prototype.New === 'undefined') {
    	Function.prototype.New= function(...args) {
    		// create instance 
    		// inst can be either { f1, f2 } or { ctor: f1, header: { f2, f3 } }
    		let inst=Reflect.construct(this, []); 
    
    		// get header and ctor from inst
    		if (!inst || typeof inst!=='object' || Array.isArray(inst) || typeof inst==='function' || Object.keys(inst).length===0) throw 'New - invalid header';
    		let header, ctor=inst.ctor;
    		if (ctor && typeof ctor!=='function') throw 'New - invalid header';
    		header=(ctor ? inst.header : inst);    
    		if (!header || typeof header!=='object' || Array.isArray(header) || typeof header==='function' || Object.keys(header).length===0) throw 'New - invalid header';
    		if (header.ctor) throw 'New - invalid header';
    		Object.setPrototypeOf(header, this.prototype); // fix prototype for instanceof
    		
    		// call ctor
    		if (args.length>0 && !ctor) throw('New - missing ctor'); // no ctor to send arguments
    		if (ctor) ctor(...args); 
    
    		return header;
    	}
    }

**Simple class - no constructor or attributes:**

    function Counter() {
    	// private variables & methods
    	let count=0;
    
    	function next() {
    		return ++count;
    	}
    	
    	function reset(newCount) {
    		count=newCount;
    	}
    	
    	function value() {
    		return count;
    	}
    
    	// public interface
    	return {
    		next,  // get next value
    		reset, // reset value
    		value  // get value
    	}
    }
    	
    let counter=Counter.New();
    console.log(counter instanceof Counter);
    console.log('Counter next = '+counter.next());
    counter.reset(100);
    console.log('Counter next = '+counter.next());

**Complete class - with constructor, attributes & static methods:**

    function ColoredDiv() {
    	// private variables & methods
    	let elem;
    	let state; // true=red, false=blue
    
    	// create static instance counter
    	if (!ColoredDiv.staticCounter) ColoredDiv.staticCounter=Counter.New();
    
    	function toggle(newState) {
    		let oldState=state;
    		if (typeof newState==='undefined') state=!state; else state=newState;
    		elem.style.color=(state ? 'red' : 'blue');
    	}
    
    	function red() {
    		toggle(true);
    	}
    	
    	function blue() {
    		toggle(false);
    	}
    	
    	// constructor
    	function ctor(elem_, state_=true) {
    		console.log('ctor');
    		elem=elem_;
    		state=state_
    
    		elem.onclick = e => toggle() ;
    		
    		toggle(state_);
    		
    		// update static instance counter
    		ColoredDiv.staticCounter.next();
    	}
    	
    	// static methods
    	ColoredDiv.NumInstances = function() {
    		return ColoredDiv.staticCounter.value();
    	}
    
    	// public interface
    	let header = {
    		red,  // color elem red
    		blue, // color elem blue
    		get state() { return state; },
    		set state(s) { toggle(s); }
    	};
    	
    	return { ctor, header };
    }
    
    let myDiv1=document.getElementById('myDiv1');
    let coloredDiv = ColoredDiv.New(myDiv1);
    console.log(coloredDiv instanceof ColoredDiv);
    coloredDiv.blue();
    
    let myDiv2=document.getElementById('myDiv2');
    let coloredDiv2 = ColoredDiv.New(myDiv2, false);
    setTimeout( () => {
    	coloredDiv2.state=true;
    }, 1000);
    console.log(ColoredDiv.NumInstances());

## Caviets ##

 - The way the code is ATM there is not much support for inheritance, but I'm sure this can be added quite easily. Personally I am trying to avoid inheritance in favor of composition.

## Example ##

See a running example at [plunkr](https://plnkr.co/edit/aLp6Jj1MAUo8qBM7GvPs)




