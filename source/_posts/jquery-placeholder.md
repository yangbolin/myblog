title: 使用jQuery来模拟input的placeholder
date: 2014-04-13 17:30:00
tags: jQuery
categories: 前端开发
---

####一.需求描述
我们在写HTML代码的时候后，经常遇到input元素，在使用input元素的时候，我们有时需要利用input的placeholder属性来告诉用户在这个输入框中应该输入什么内容，提升用户体验。不幸的是，IE系列的浏览器不支持placeholder属性，那么要想在IE系列的浏览器中提升用户体验，只能利用jquery来模拟了。

<!-- more -->

####二.使用jQuery模拟placeholder代码
```javascript
if(!$.support.placeholder) {
    // 使用选择器，选中所有带有placeholder属性的input元素列表
	var els = $('input[placeholder],textarea[placeholder]');
	// 遍历每个带有placeholder属性的元素
	els.each(function(i, el) {
		el = $(el);
		// 获取placeholder的值以及颜色值
		var defValue = el.attr('placeholder'),
			defColor = el.css('color');
		// 当输入框获取到焦点后，把placeholder的值清空
		el.bind('focus',function() {
			if(this.value === '' || this.value === defValue) {
				$(this).css('color', defColor);
				this.value = '';
			}
		});
		// 当输入框失去焦点后，要是用户没有输入值，或者输入的值和placeholder的值一样，就把placeholder属性再次展示出来
		el.bind('blur',function() {
			if(this.value === '' || this.value === defValue) {
				$(this).css('color','#aaa');
				this.value = defValue;
			}
		});
		el.triggerHandler('blur');
        // 在表单提交时，需要判断用户输入的值是否和placeholder的值一样，要是一样就需要清空
		el.closest('form').submit(function(){
			var val = el.val();
			if(val === defValue){
				el.val('');
			}
		});
	});
}
```