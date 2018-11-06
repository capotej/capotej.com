---
layout: post
title: "Arrow key navigation for text fields"
date: 2008-10-12T17:41:00Z
comments: false
permalink: /post/54266325/arrow-key-navigation-for-text-fields
tags:
---



Here is a class for enabling the use of arrow keys to navigate through a grid of input fields: (using mootools)

```javascript
var FocusMover = new Class({

        initialize: function(sel, col_num){

                this.sel = sel
                this.col_num = col_num
                this.inputs = $$(this.sel)
                this.current_focus = 0

                var self = this

                this.inputs.each(function(item, index){
                        item.addEvent('keydown',function(key){
                                $try(function(){
                                        self[key.key]()
                                })
                        })
                        item.addEvent('focus',function(e){
                                self.refresh(e)
                        })

                        item.set('myid', index)
                })

                this.inputs[0].focus()

        },


        refresh: function(e){
                this.current_focus = e.target.get('myid')
        },

        down: function(){
                i = parseInt(this.current_focus) + parseInt(this.col_num)
                this.inputs[i].focus()
        },

        up: function(){
                i = parseInt(this.current_focus) - parseInt(this.col_num)
                this.inputs[i].focus()
        },

        left: function(){
                i = parseInt(this.current_focus) - 1
                this.inputs[i].focus()
        },

        right: function(){
                i = parseInt(this.current_focus) + 1
                this.inputs[i].focus()
        }

})
```

As you can see, the constructor takes two arguments: a selector (which should return a list of all your input fields), and the number of input field columns. So for a 4x2 table, you would set it up like this:

```javascript
var FM = new FocusMover('#mytable input', 4)
```
