# Webpack另类扣法(半自动吐)

项目地址 : **aHR0cHM6Ly93ZWIuZXd0MzYwLmNvbS9yZWdpc3Rlci8jL2xvZ2luP19rPXJxc2Z0ag==**

## 第一步(观察加密参数特征)

老规矩, 密码123456

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_275c188efb422d83adf2c05427a8b5c3.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_275c188efb422d83adf2c05427a8b5c3.jpg)

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_c32376582d497a09d0db3ce01c3552ba.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_c32376582d497a09d0db3ce01c3552ba.jpg)
长度32位, 像这种密码之类的加密, 相信我, 一般不太会是hash系列的加密 ,我们看发包的调用栈, 找找加密函数的位置

## 第二步(查看加密函数位置)

像一些login的关键词, 我们就可以大胆的点进去看看, 

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_72f79b5ed37f4a6ffcb21570883d83da.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_72f79b5ed37f4a6ffcb21570883d83da.jpg)
很轻松的就找到对应的加密函数, g.Encrypt , 这里我们就需要找到g的位置, 没有g哪来的Encrypt

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_1394eea63f268c42e454912fc09fb9e6.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_1394eea63f268c42e454912fc09fb9e6.jpg)

找到对应的g函数, 因为该站用了webpack , 所以n是个加载器, 懂webpack扣法的朋友应该没问题, 这里就引出我们今天的主题了, 以往的扣法都是一步一步的扣, 有n的函数都抠下来, 但我们今天让它自己吐出来, 嘿嘿

## 第三步(扣代码)

我们先断在g处 , 此时还没有调用加载器 

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_22bf3986fcd8d2ed3d62b5b7ad31080e.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_22bf3986fcd8d2ed3d62b5b7ad31080e.jpg)

进入g函数, 在e[r]处打上断点 , 跟进来, 此时可以发现e是一系列函数数组, 而r就是410 , 我们此时要做的操作来啦 

```javascript
window.zhiyuan = D; // 该处的D是加载器,不同的网页有不同的参数,自己参考
window._wbpk =  r.toString()+":"+(e[r]+"")+ ","; // 这里是拿到对应的键值, 类似 410 : 函数.toString() , 利用这个手段
// 就达到自己吐函数的目的了

// hook加载器, 让它每调用一次加载器,就记录一次值,尽管可能每个函数里再调用其他函数,没关系,依次hook, 切记该hook的位置很关键,上面已经说了, 一定要在我们加密参数之前, 
window.cache = {}
D = function(r){
    // 这里其实就是一个缓存,如果我们已经存过该值了,就没必要再存一遍了
    if(!window.cache[r]){
		window.cache[r] = true;
        window._wbpk = window._wbpk + r.toString()+":"+(e[r]+"")+ ",";    	
    }
    return window.zhiyuan(r);

}
```

**大声说: 志远大佬牛逼, 跟着志远大佬吃香喝辣**

因为这里的操作很关键 , 允许我再复述一遍

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_ebb38a1d3dd9db52b275694325faec8e.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_ebb38a1d3dd9db52b275694325faec8e.jpg)

先在加密处g打上断点, y处也打上, 为什么呢? 因为我们hook掉加载器后, 直接过到y, 以便我们的代码全部吐出来了

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_8a6c97bf198adedadcdaeeb0e5074134.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_8a6c97bf198adedadcdaeeb0e5074134.jpg)

在e[r]处打上断点, 跟过来, 当r显示对应值的时候 ,输入我们的hook代码

然后把e[r]的断点放掉, 为什么呢? 因为我们不敢保证g调用加载器的时候是否有其它的函数调用, 如果有的话,那它会一直来到e[r]

我们不想要它的过程, 让它直接加载完

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_3627e6a7ebfe2df436493d267036f2c4.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_3627e6a7ebfe2df436493d267036f2c4.jpg)
[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_e7b6b0ad69a799cfc0ccf103c48484d8.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_e7b6b0ad69a799cfc0ccf103c48484d8.jpg)
已经把我们需要的依赖函数全部吐出来了, 真不错 , 老规矩 

组装代码

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_692bd330aed1d02d7caa9d85d7756774.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_692bd330aed1d02d7caa9d85d7756774.jpg)
这里把我们要调用的加密导出来, 就可以在外部使用了

[![](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_e2532658b1c8b2b6baff253187a8b2b7.jpg)](https://www.heqiang.site/wp-content/uploads/2020/10/wp_editor_md_e2532658b1c8b2b6baff253187a8b2b7.jpg)

大功告成 , 溜了