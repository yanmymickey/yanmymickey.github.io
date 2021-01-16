---
title: JavaScript 模板
date: 2019-07-15
tags: [Note,javascript]
categories:
- [Note,JavaScript]
comments: false
---
#   JavaScript 模板

<!-- more -->

    //获取父元素  
    row.parentNode.parentNode.parentNode.removeChild(row.parentNode.parentNode);  
           
    //获取子元素  
    tr2[n].firstElementChild  
           
    //input值获取  
    var x=document.getElementById("name").value  
           
    //radio选择获取  
    var y=document.getElementsByName("sex");  
    var sexname = null;  
    for(var m=0;m<y.length;m++){  
    	if(y[m].checked){  
        	sexname = y[m].value;  
        }  
    }  
          
    //inner HTML格式  
    innerHTML="<td>"+n+"</td>"+<td><a href='#' onclick='deleteTr(this)'>删除</a></td>"            
    //字符串（标签）  
    innerText = n                           //变量  
          
    //alert格式  
    alert("hello world")                 //字符串  
    alert(y）                                  //变量  
          
    //一堆同类型标签绑定事件  
    function change () {  
    	var td = document.getElementsByTagName("td");  
    	for(var i=0;i<td.length;i++){  
    		if(i>3){  
    			td[i].onmouseover = function() {  
    			this.style.color = "red";  
    		}  
       		td[i].onmouseout = function() {  
                this.removeAttribute("style");                                  
    		}  
    	}  
    }  
