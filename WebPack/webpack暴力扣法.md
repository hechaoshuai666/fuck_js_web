# webpack暴力扣法

项目地址 : **aHR0cHM6Ly93d3cuZ205OS5jb20v**

## 第一步(观察加密参数特征)

老规矩, 密码123456

![image-20201103152907164](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103152907164.png)

看到加密后的特征, 根据笔者的经验,一般来说密码之类的东西,大部分使用的是AES , RSA等加密, RSA比较多, 像信息摘要之类的,就保证数据的一致性,会使用Hash系列的签名算法,这玩意就靠多弄

我们看发包的调用栈, 找找加密函数的位置

## 第二步(查看加密函数位置)

推荐大家使用调用栈来查找加密函数的位置,为什么呢? 有的时候我们可以通过查找关键词快速定位, 但这个方法却不是万能的, 这里推荐使用调用栈,一方面是锻炼快速查找加密函数的能力,另一方面是多积累查看代码的运行的逻辑,有助于秃头

像login啥的关键词,直接点进去看

![image-20201103153352855](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103153352855.png)

很快就定位到对应的加密位置,是通过g.encode()返回的结果,弄这玩意带有很强的目的性,我不注重你加密的过程,我只要你加密的结果,所以我们也只要它加密的结果

大致流程就是明文->加密函数->发包,我们要的就是加密函数,管它中间有多少逻辑呢

既然是g.encode进行加密,那我们进到该方法里面看看加密逻辑咯

![image-20201103155755881](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103155755881.png)

可以看到其实加密的真正方法是this.jsencrypt.encrypt, 进入该函数康康

![image-20201103155936452](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103155936452.png)

是一段很长的js代码,这里因为篇幅有限,就不展示了,我们把这段代码给抠下来,要注意作用域的问题

这里就引出我们今天的主题了,webpack!!!

webpack打包后的工作流程如下

```javascript
!function(i) {
    var s = {};
    function n(t) {
        if (s[t])
            return s[t].exports;
        var e = s[t] = {
            i: t,
            l: !1,
            exports: {}
        };
        return i[t].call(e.exports, e, e.exports, n),
        e.l = !0,
        e.exports
    }
}([function(){},xxx,xxx])

//首先是一个自执行函数,括号里面放了webpack打包后的函数数组,当用到某一个函数时,可通过n(数组索引)来加载对应的函数,上面的代码很清晰了,稍微看下就能明白,自执行里面n最终会被导出为全局变量,方便需要的使用调用并加载函数,这里的n是加载器(特别重要)

//所以当你看到有个自执行函数,且里面有类似于上述的结构,加载器的名字不一定是n,(i[t].call(e.exports, e, e.exports, n),e.l = !0,e.exports)等特征,且调用参数是一个数组,那么就是webpack了
```

![image-20201103154409658](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103154409658.png)

![image-20201103154418435](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103154418435.png)

可以看到这个网站就是用webpack打包的,除了加载器n的函数,其它的逻辑对我们来说都不重要,在扣代码的时候可以删掉

## 第三步(扣代码)

#### 构造自执行函数

因为网站已经有加载器和自执行函数了,我这里就直接拿过来用了

![image-20201103161853370](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103161853370.png)

webpack打包用的是数组,我们这里用对象,因为我们的内容少,且对象调用的时候比较方便

上面的encrypted就是this.encrypted函数进去的代码,我把它抠下来了,还有一步

![image-20201103162057631](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103162057631.png)

调用该方法的函数我们也要扣下来,会发现里面有个r(3),其实这里的r就是我们的加载器,可以看上面的webpack示例代码,写的很详细,我们把这个代码也给抠下来,

![image-20201103162738163](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103162738163.png)

最后的结果就是这样,还有一点小细节没讲到,前面已经说过了r就是我们的加载器,可以看到var s = r(3),这里的相当于取数组的第四个函数,细心的你一定可以发现这个函数就是'encrypted',所以我们把它扣出来后也要改下,把r(3)改成r('encrypted'),还有加载器n也要导出来,不导出来我们没法调用,最终版

![image-20201103163135315](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103163135315.png)

我们要把对应的接口给导出来,然后根据网站上怎么加密,我们就怎么用,可以看到n.prototype.encode,这里用到了原型链(且里面有this),所以我们要new,才确保用encode的时候不出错

![image-20201103163355956](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103163355956.png)

结束了

番外话,希望你们能懂!!!

其实上面讲的逻辑有点乱,相当于倒序了,正常来说不是这样扣的,因为这个网站比较简单,所以一下子能解决,正常流程应该是如下操作的

我们首先看到g.encode, 我们要找到g对象

![image-20201103163634277](C:\Users\WilliamWeson\AppData\Roaming\Typora\typora-user-images\image-20201103163634277.png)

是不是有点熟悉(new的方法?),n(2)其实就是我们的decrypted,真正的扣法其实是先从g下手,发现是从n(2)调用来的,再去扣n2,扣到n2的时候会发现有个r(3),r(3)就是encrypted函数,一步一步下去,直到没有调用加载器为止,你可以自己去扣代码康康,看看顺序是不是这样,我前面讲的直接找到了最后一步,再慢慢回溯

加载器的名字不是固定的,一般来说都是最后一个参数

有些网站特别恶心,你在扣这个依赖函数的时候,里面还有很多加载器的调用,你会发现越扣越累,可以去看看我有一篇半自动扣的,可以解决一部分网站

这篇博客码了两小时,害, 主要是我按老师给我讲的思路来的,我自己的思路不是这样,所以卡住了,我原本想直接通过hook n(2)拿到所有的函数, 因为这个js是动态变化的,没法下断点,用fiddler替换的时候又报跨域了,头大,就一点一点的看了

讲的不好,献丑了