# webpack另类扣法(半自动二)

项目地址:**aHR0cHM6Ly9zdGF0aWMud2FpdHdhaXRwYXkuY29tL3dlYi9zZF9zZS9pbmRleC5odG1sIy9zZWFyY2gvc2VhcmNoZm9yPXZlbmRvciZrZXl3b3JkPSVFNyVCMSVCMyVFNyVCMiU4OQ==**

前面写过一篇半自动扣的文章了,但当时用的手段是在加密函数前hook加载器,然后跳过加密函数,直接拿到所有的依赖函数,但有些站则不一样,如果你研究过加载器的写法,你会发现它里面有个缓存器,也就是加载过的函数直接从缓存里拿,有的时候我们hook加密函数,它里面有些依赖函数早就执行了,我们再hook加载器,加载过的不会再hook,这就导致我们hook的代码不全

典型的报错就是undefined无法调用call方法,看到这个错误提示,那么一定是hook的代码不全

经过笔者的冥思苦想,目前发现了两种情况

- 如果加载器跟函数列表不是同一个js文件,那么很有可能加密函数只有在加密的时候才会调用,也就是不会有缓存的存在,我们在加密函数前hook的手段可能有效
- 如果加载器跟函数列表是同一个js文件,那么很有可能加密函数内的依赖在加载某个函数的时候已经记载过了,所以加密函数调用的时候里面有些依赖函数已经缓存过了,导致我们hook不全

这是目前笔者所遇到的两种情况,具体情况先按加密前hook,然后调用下看看缺不缺函数,缺的话再想办法

今天的主题就是加载器跟函数列表在同一个js文件里,而且加载加密函数前某些依赖函数已经被call了

## 第一步(观察加密参数特征)

![image-20201103180027827][![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_8f928fcb13ea468a35b514913fba1f30.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_8f928fcb13ea468a35b514913fba1f30.jpg)

这个是响应数据加密,看这一大串数据,也看不出个所以然,管它呢, 看调用栈, 如果是xhr,可以通过xhr断点一步一步往栈底走

笔者这里直接用xhr断点了

## 第二步(查找加密函数位置)

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_7a950abb08d09d1486883f428c9e2266.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_7a950abb08d09d1486883f428c9e2266.jpg)

断下来,是在异步函数里调用, 可真是操蛋, 管它呢,先单步进入康康

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_7864397fc96c65b7d011d606f86ee23e.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_7864397fc96c65b7d011d606f86ee23e.jpg)
目前看来好像要进入f函数里,而且e就是响应回来的加密数据

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_6e00e63e5c6a55c465398093054c9d9b.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_6e00e63e5c6a55c465398093054c9d9b.jpg)

可以确定f就是解密函数了

此时我们的头脑一定要十分清楚自己想要干嘛, 既然f是解密函数,那么我们要找到f函数所在的作用域, 因为该站是webpack, 找到解密函数所在的大函数, 一定是有地方加载了这个大函数,且把f给导出来了,不然怎么直接调用的f

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_01900b1ac38083a85786f95930093fa0.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_01900b1ac38083a85786f95930093fa0.jpg)

可以看到在下面果然把f给导出去了,就是decodeData, 回到我们进入异步的地方,看调用栈,一步一步的查找调用该大函数的地方,然后hook

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_501a713264270ff503d431717cf28217.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_501a713264270ff503d431717cf28217.jpg)

这段解密调用是在fetchData里进行的, 我们要抓住几个特征,好查找

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_c0056ac09bd28dacb2b5246ba5da804e.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_c0056ac09bd28dacb2b5246ba5da804e.jpg)

看调用栈的时候发现有个a.fetchData,这不就是我们之前看的解密调用吗? 往上面看看a是怎么来的?

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_e5ab69767f4166d9ab06a5a9063c7ac6.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_e5ab69767f4166d9ab06a5a9063c7ac6.jpg)

嘿嘿, 好家伙,n熟悉的加载器呀, 之前我就在a,u处下断点, hook加载器, 结果hook的代码不全, 想了很多办法, 把缓存给干掉,但都发现不妥

后面我直接在加载器call的地方下断点, 你不是之前就调用了某些依赖函数吗? 我给你断住,自己执行我要的函数,这不就o了吗,想法不错,下课后我就试了下,果然可以

## 第三步(扣函数)

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_ae94f6820df3adaddea366e1aff707ee.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_ae94f6820df3adaddea366e1aff707ee.jpg)

有我们想要的解密函数,且t对象为空, 因为此时还没有加载任何依赖函数, 我们新起一个snippet,然后在里面写我们的hook代码

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_e6c9cbdce62621373a6cc9719e8a01a5.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_e6c9cbdce62621373a6cc9719e8a01a5.jpg)

已经执行成功了

[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_7db11846589e239a5b5ba868a6fa12c4.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_7db11846589e239a5b5ba868a6fa12c4.jpg)

删掉前面的undefined, 然后把代码封装下,按照之前讲的, 这里我就不演示了, 缺啥补啥, setTimeout, navigator之类的, 后面就可以通过调用n(29)获取对象,再decodeData就完事了
[![](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_0bda395fa6661f92acc325560f97017b.jpg)](https://www.heqiang.site/wp-content/uploads/2020/11/wp_editor_md_0bda395fa6661f92acc325560f97017b.jpg)
该方法不敢保证每个情况都合适, 根据不同的网站写不同时机的hook代码就行了

溜了,peace