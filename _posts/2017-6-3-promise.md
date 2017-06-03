---
layout: post
title: Promise初识之-介绍
categories: ES6
description: ES6的 Promise介绍
keywords: promise, 异步编程
---

## Promise 语法

    new Promise(
    /* executor */
    function(resolve, reject) {...}
	);
	
executor是一个带有resolve和reject两个参数的函数 。executor 函数由Promise的实现立即执行，
传递resolve和reject函数。resolve 和 reject 函数，当被调用时，分别解决或拒绝 promise。
executor 通常会启动一些异步工作，然后，一旦完成，
可以调用resolve函数来解决promise，否则在发生错误时拒绝它。
如果在executor函数中抛出一个错误，那么该promise 将被拒绝。executor的返回值被忽略。

## Promise 描述

Promise 对象是一个代理对象（代理一个值），被代理的值在Promise对象创建时可能是未知的。
它允许你为异步代码执行结果的成功和失败分别绑定相应的处理方法（handlers ）。 
这让异步方法可以像同步方法那样返回值，但是并非立即返回执行的结果，因为毕竟执行的是异步代码，
因此，它会返回一个Promise对象，如前所说，它是一个代理的对象，代理了最终返回的值，可以在后期使用
。

## Promise的几种状态
- pending: 初始状态，未履行或拒绝。
- fulfilled: 意味着操作成功完成。
- rejected: 意味着操作失败。

pending 状态的 Promise 对象可能以 fulfilled 状态返回了一个值，也可能被某种理由（异常信息）拒绝（reject）了。
当其中任一种情况出现时，Promise 对象的 then 方法绑定的处理方法（handlers ）就会被调用（then方法包含两个参数：
onfulfilled 和 onrejected，它们都是 Function 类型。当值被填充时，调用 then 的 onfulfilled 方法，
当Promise被拒绝时，调用 then 的 onrejected 方法， 所以在异步操作的完成和绑定处理方法之间不存在竞争）。

一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise 对象的状态改变，只有两种可能：从 Pending 变为 fulfilled 和从 Pending 变为 Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对 Promise 对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

## Promise用法

JavaScript是可以执行异步操作的，所以就会存在异步回调函数，比如

    function aa(callback1) {
        //执行异步操作
        if (xxx) {
            callback1('a1');
        }	
    }

咋一看这么写没什么问题，异步操作执行完后，再执行回调函数，但是，要是callback1也是个异步函数怎么办，上面的代码变成：
	
	function callback1(v1) {
        //执行异步操作
        if (xxx) {
            callback2(v2)
        }
    }
    
    function callback2(v2) {
        //执行xxx
    }
 
    function aa() {
        //执行异步操作
        if (xxx) {
            callback1('v1');
        }	
    }

每加一个回调，就会多一层嵌套，嵌套越来越多，如此的代码后面维护起来就比较困难，回调函数就会进入无底洞，
而Promise可以解决这个问题：

    function callback1() {
        var p2 = new Promise(function(resolve, reject){
             //执行异步操作
             if (xxx) {
                 resolve('v2');
             } else {
                 reject('v2');
             }  
        });
        
        return p2;
    }
    
    function callback2() {
        var p3 = new Promise(function(resolve, reject){
             //执行异步操作
             if (xxx) {
                 resolve('v3');
             } else {
                 reject('v3');
             }  
        });
 
        return p3;
    }

    function aa() {
        var p1 = new Promise(function(resolve, reject){
            //执行异步操作
            if (xxx) {
                resolve('v1');
            } else {
                reject('v1');
            }        
        });
        
        return p1;
    }
    
    aa().then(function(){
       return callback1();
 
    }).then(function(){
        return callback2();
 
    });
    
显然，代码这么写比原来好看多了，且能把执行代码跟结果处理分开了。


## Promise的方法

#### Promise.all()

Promise.all(iterable) 方法指当所有在可迭代参数中的 promises 已完成，或者任何一个传递的 promise（指 reject）失败时，
返回 promise。参数iterable是一个promise数组

Promise.all 是当所有给定的可迭代完成时执行 resolve，或者任何  promises 失败时执行 reject。

如果传递任何的 promises rejects ，所有的 Promise 的值立即失败，丢弃所有的其他 promises，如果它们未 resolved。
如果传递任意的空数组，那么这个方法将立刻完成。

Promise.all 等待所有代码的完成（或第一个代码的失败）。

    var p1 = new Promise(function(resolve, reject){
        setTimeout(resolve('v1'), 100);
    });
    var p2 = new Promise(function(resolve, reject){
        setTimeout(resolve('v2'), 200);
    });
    
    Promise.all([p1, p2]).then(function(values){
        console.log('resolve');
        console.log(values);
  
    }, function(values){
        console.log('reject');
        console.log(values);

    });

可以看到控制台会输出一个数组

    resolve
    ["v1", "v2"]
    
当有一个被reject时，修改一下上面的代码

     var p1 = new Promise(function(resolve, reject){
        setTimeout(reject, 500, 'v1');
     });
    var p2 = new Promise(function(resolve, reject){
        setTimeout(reject, 200, 'v2');
    });
    
    Promise.all([p1, p2]).then(function(values){
        console.log('resolve');
        console.log(values);
   
    }, function(values){
        console.log('reject');
        console.log(values);
 
    });
    
浏览器输出如下

    reject
    v2
    
再修改代码如下：

    var p1 = new Promise(function(resolve, reject){
        setTimeout(resolve, 100, 'v1');
     });
    var p2 = new Promise(function(resolve, reject){
        setTimeout(reject, 500, 'v2');
    });
    var p3 = new Promise(function(resolve, reject){
        setTimeout(reject, 200, 'v3');
    });
    
    Promise.all([p1, p2, p3]).then(function(values){
        console.log('resolve');
        console.log(values);
   
    }, function(values){
        console.log('reject');
        console.log(values);
 
    });
    
浏览器输出如下

    reject
    v3

Proimise.all()当传入的promise数组中<font color="red">有一个代码的reject</font>时就会立即结束。

#### Promise.race()

Promise.race(iterable) 方法返回一个 promise，在可迭代的 resolves 或 rejects 中 promises 有一个完成或失败，
将显示其值或原因。参数iterable是一个数组。

race 函数返回一个 Promise，它将与第一个传递的 promise 相同的完成方式被完成。它可以是完成（ resolves），
也可以是失败（rejects），这要取决于第一个完成的方式是两个中的哪个。

     var p1 = new Promise(function(resolve, reject){
        setTimeout(resolve, 500, 'v1');
     });
    var p2 = new Promise(function(resolve, reject){
        setTimeout(reject, 400, 'v2');
    });

    Promise.race([p1, p2]).then(function(values){
        console.log('resolve');
        console.log(values);

    }, function(values){
        console.log('reject');
        console.log(values);

    });
    
p2比p1执行快，所以浏览输出为:

    reject
    v2
 
修改为
    
    var p1 = new Promise(function(resolve, reject){
         setTimeout(resolve, 300, 'v1');
      });
    var p2 = new Promise(function(resolve, reject){
        setTimeout(reject, 400, 'v2');
    });
    
     Promise.race([p1, p2]).then(function(values){
         console.log('resolve');
         console.log(values);
    
     }, function(values){
         console.log('reject');
         console.log(values);
    
     });
     
p1比p2执行快，所以浏览输出为:

    resolve
    v1
    
Promise.race是谁先执行完，就返回谁的状态resolve或者reject


#### Promise.prototype.catch()

catch() 方法返回一个Promise，只处理拒绝的情况。它的行为与调用Promise.prototype.then(undefined, onRejected) 相同。

catch 方法可以用于您的promise组合中的错误处理。

当Promise.prototype.then方法传入了onRejected参数时，catch方法不会执行

    var p1 = new Promise(function(resolve, reject){
         setTimeout(reject, 300, 'v1');
    });
     
    p1.then(function(values){
        console.log('resolve');
        console.log(values);
      
    }, function(values){
        console.log('rejct');
        console.log(values);
 
    }).catch(function(e){
        console.log("捕抓reject====" + e);
    });
    
浏览器输出：

    rejct
    v1
    
去掉onRejected参数

    var p1 = new Promise(function(resolve, reject){
         setTimeout(reject, 300, 'v1');
    });
     
    p1.then(function(values){
        console.log('resolve');
        console.log(values);
      
    }).catch(function(e){
        console.log("捕抓reject====" + e);
    });

浏览器输出

    捕抓reject====v1
    
进入了catch调用后，调用链就会被重置

    var p1 = new Promise(function(resolve, reject){
        setTimeout(resolve, 300, 'v1');
    });

    p1.then(function(values){
        console.log('resolve');
        console.log(values);
        
        return Promise.reject('catch reject')

    }).catch(function(e){
        console.log("捕抓reject====" + e);
        
    }).then(function(values){
        console.log("resolve====" + values);

    }, function(values){
        console.log("reject====" + values);

    });
 
浏览器输出

    resolve
    v1
    捕抓reject====catch reject
    resolve====undefined
    
第二个then输出了"resolve===="，而不是"reject===="，证明了如上说法，而没有进入catch的则不会，如下代码

    
    var p1 = new Promise(function(resolve, reject){
        setTimeout(resolve, 300, 'v1');
    });

    p1.then(function(values){
        console.log('resolve');
        console.log(values);
        
        return Promise.resolve('catch reject')

    }).catch(function(e){
        console.log("捕抓reject====" + e);
        
    }).then(function(values){
        console.log("resolve====" + values);

    }, function(values){
        console.log("reject====" + values);

    });
    
 浏览器输出如下：
 
    resolve
    v1
    resolve====catch reject