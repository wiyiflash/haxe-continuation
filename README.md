haxe-continuation
=================

An *asynchronous functions* is a function that accept its last parameter 
as a callback function.
And **haxe-continuation** is a macro library enables you to invoke and write
asynchronous functions like a synchronization function, and automatically
transform the function into *continuation-passing style (CPS)*. That means
you can write code looks like *multithreading* without platform
multithreading support.

## Installation

I have upload haxe-continuation to haxelib. To install, type the following
command in shell:

    haxelib install continuation

Now you can use continuation in your project:

Output to JavaScript:

    haxe -lib continuation -main Your.hx -js your-output.js

, or output to SWF:

    haxe -lib continuation -main Your.hx -swf your-output.swf

, or output to any other platform that Haxe supports.

haxe-continuation requires Haxe 2.10 or Haxe 3.

## Usage

To write a CPS function, put `@:build(com.dongxiguo.continuation.Continuation.cpsByMeta(":cps"))`
before a class, and marking the CPS functions in that class as `@:cps`:

    import com.dongxiguo.continuation.Continuation;
    @:build(com.dongxiguo.continuation.Continuation.cpsByMeta(":cps"))
    class Sample
    {
    
      // An asynchronous function without automatically CPS transformation.
      static function sleepOneSecond(handler:Void->Void):Void
      {
        haxe.Timer.delay(handler, 1000);
      }
    
      // Because of the magic @:cps, this function Will be transform to:
      // static function asyncText(__return:Void->Void):Void
      @:cps static function asyncTest():Void
      {
        trace("Start continuation.");
        for (i in 0...10)
        {
          // Magic .async() postfix to invoke an asynchronous function.
          sleepOneSecond().async();
          trace("Run sleepOneSecond " + i + " times.");
        }
        trace("Continuation is done.");
      }
    
      public static function main() 
      {
        asyncTest(function()
        {
          trace("Handler without continuation.");
        });
      }
    
    }

In CPS functions, `async` is a magic word to invoke other
async functions. When calling an asynchronous function with the `.async()` postfix, you need not to explicitly pass a callback
function. Instead, the code after `.async()` will be captured as the callback
function for the callee.

Another way is using `Continuation.cpsFunction` macro to write nested CPS functions:

    import com.dongxiguo.continuation.Continuation;
    class Sample2
    {
      // An asynchronous function without automatically CPS transformation.
      static function sleepOneSecond(handler:Void->Void):Void
      {
        haxe.Timer.delay(handler, 1000);
      }
      public static function main() 
      {
        // This magic macro will transform function asyncTest to:
        // function asyncText(__return:Void->Void):Void
        Continuation.cpsFunction(function asyncTest():Void
        {
          trace("Start continuation.");
          for (i in 0...10)
          {
            // Magic .async() postfix to invoke an asynchronous function.
            sleepOneSecond().async();
            trace("Run sleepOneSecond " + i + " times.");
          }
          trace("Continuation is done.");
        });
        asyncTest(function()
        {
          trace("Handler without continuation.");
        });
      }
    }


See https://github.com/Atry/haxe-continuation/blob/master/tests/TestContinuation.hx
for more examples.

### Working with [hx-node](https://github.com/cloudshift/hx-node)

Look at https://github.com/Atry/haxe-continuation/blob/master/tests/TestNode.hx.
The example forks 5 threads, and calls Node.js's asynchronous functions in each thread.

### Generator

haxe-continuation also provider an utility to wrap CPS functions into `Iterator`s.

For example:

    using com.dongxiguo.continuation.utils.Generator;
    @:build(com.dongxiguo.continuation.Continuation.cpsByMeta(":cps"))
    class TestGenerator
    {
      @:cps
      static function intGenerator(yield:YieldFunction<Int>):Void
      {
        for (i in 1...4)
        {
          for (j in 1...(i+1))
          {
            trace('$j * $i =');
            yield(i * j).async();
            trace("-------");
          }
        }
      }
      public static function main() 
      {
        for (i in intGenerator)
        {
          trace(i);
        }
      }
    }

The output:

    TestGenerator.hx:47: 1 * 1 =
    TestGenerator.hx:59: 1
    TestGenerator.hx:49: -------
    TestGenerator.hx:47: 1 * 2 =
    TestGenerator.hx:59: 2
    TestGenerator.hx:49: -------
    TestGenerator.hx:47: 2 * 2 =
    TestGenerator.hx:59: 4
    TestGenerator.hx:49: -------
    TestGenerator.hx:47: 1 * 3 =
    TestGenerator.hx:59: 3
    TestGenerator.hx:49: -------
    TestGenerator.hx:47: 2 * 3 =
    TestGenerator.hx:59: 6
    TestGenerator.hx:49: -------
    TestGenerator.hx:47: 3 * 3 =
    TestGenerator.hx:59: 9
    TestGenerator.hx:49: -------

## License

Copyright (c) 2012, 杨博 (Yang Bo)
All rights reserved.

Author: 杨博 (Yang Bo) <pop.atry@gmail.com>

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice,
  this list of conditions and the following disclaimer.
* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.
* Neither the name of the <ORGANIZATION> nor the names of its contributors
  may be used to endorse or promote products derived from this software
  without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
