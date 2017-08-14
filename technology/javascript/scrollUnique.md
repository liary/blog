# 张鑫旭子元素滚动父元素不跟随滚动JS实现优化
> 最近处理了一个bug，表现为一个弹窗里的下拉菜单不能滚动，但弹窗body可以滚动，原因是弹窗引用了一个子元素滚动父元素容器不跟随滚动但方法，方法基于张鑫旭的[小tip: 子元素scroll父元素容器不跟随滚动JS实现](http://www.zhangxinxu.com/wordpress/2015/12/element-scroll-prevent-parent-element-scroll-js/)，由于弹窗注册了这个方法，在弹窗里的所有滚轮事件都进入了注册元素的滚动判断里，引发了问题。

## 改进思路
1. 为注册了该方法的元素加一个标志，如`!$(this).hasClass('scroll-unique-mark') && $(this).addClass('scroll-unique-mark')`
2. 再为注册的方法加一个独立的hash作为唯一身份标记
3. 滚轮事件触发时，判断当前事件的target元素最近的注册元素`$(e.target).parents('.scroll-unique-mark').eq(0)`或`$(e.target) //本身就是注册了本方法的元素`的hash是否与`$(this)`元素相等，相等即判断为在当前注册容器内滚动的滚动事件，否则则当作在当前注册容器里的子注册容器上的滚动事件，跳过判断逻辑
ps: 注册容器 = 注册了scrollUnique()了的元素

## 改进代码
```javascript
$.fn.scrollUnique = function() {
	return $(this).each(function() {
		var eventType = 'mousewheel', hash = new Date().getTime() + '-' + parseInt(Math.random() * 100);
		if (document.mozHidden !== undefined) {
			eventType = 'DOMMouseScroll';
		}
		!$(this).hasClass('scroll-unique-mark') && $(this).addClass('scroll-unique-mark');
		$(this).attr('scroll-unique-hash', hash);
		$(this).on(eventType, function(event) {
			var scrollTop = this.scrollTop,
				  scrollHeight = this.scrollHeight,
				  height = this.clientHeight,
				  $t = $(event.target);

			var delta = (event.originalEvent.wheelDelta) ? event.originalEvent.wheelDelta : -(event.originalEvent.detail || 0);        
			if ((delta > 0 && scrollTop <= delta) || (delta < 0 && scrollHeight - height - scrollTop <= -1 * delta)) {
				if ($t.parents('.scroll-unique-mark').eq(0).attr('scroll-unique-hash') === $(this).attr('scroll-unique-hash') || $t.hasClass('scroll-unique-mark') && $t.attr('scroll-unique-hash') === $(this).attr('scroll-unique-hash')) {
					this.scrollTop = delta > 0 ? 0: scrollHeight;
					event.preventDefault();
				}
			}        
		});
	});	
};
```
