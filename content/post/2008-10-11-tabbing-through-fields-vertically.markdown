---
layout: post
title: "Tabbing through fields vertically"
date: 2008-10-11T01:54:00Z
comments: false
url: /post/54058512/tabbing-through-fields-vertically
tags:
---



Sometimes it’s useful to switch the browser’s default tabbing behavior (left to right) to the opposite (top to bottom) when your input fields are in a grid layout instead the of the usual single column layout. Having to do this manually is a real pain, especially for large grids; So here is a solution in javascript, using mootools:

```javascript
window.addEvent('domready', function(){
    var trs = $$('#mytable tr')
    var accum = 0
    trs.each(function(tr, trindex){
        accum = trindex + 1
        tr.getChildren().each(function(td, tdindex){
            td.getChildren('input')[0].tabIndex = accum
            accum = accum + trs.length
        })
    })
})
```
