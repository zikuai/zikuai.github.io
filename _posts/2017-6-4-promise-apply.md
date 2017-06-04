---
layout: post
title: Promise应用
categories: ES6
description: ES6的 Promise应用
keywords: promise, 异步编程
---

## Promise简单封装ajax请求

我们都知道jquery的ajax请求方法的回调方法书写起来相当麻烦
    
    $.ajax({
        url: '',
        method: '',
        data: '',
        success: funciton(){},
        fail: function(){},
        error: function(){}
    });

可以看到有三个回调函数，每次都得写三次，使用Promise封装后，只需关注结果

    function ajax(method, url, data) {
        var xhr = new XMLHttpRequest();

        return new Promise(function(resolve, reject) {
            xhr.onreadystatechange = function() {
                if (xhr.readyStatus === 4) {
                    if (xhr.status === 200) {
                        resolve(xhr.responseText);
                    } else {
                        reject(xhr.status);
                    }
                }
            }

            xhr.open(method, url);
            xhr.send(data);
        });
    }

    ajax({
        url: '',
        method: '',
        data: {  }
    }).then(function (v) {
        console.log('success===' + v)
    }, function (v) {
        console.log('fail===' + v)
    });