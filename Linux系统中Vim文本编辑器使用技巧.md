# Linux系统中Vim文本编辑器使用技巧

2025-07-18 16:37·[JasonTang](https://www.toutiao.com/c/user/token/MS4wLjABAAAAGcK3SfOVVFBC8fIGFuOwJmb3B6SWDQmbKrPYjmMA0_E/?source=tuwen_detail)

**简介：**在Linux的世界里，Vim是一个非常受欢迎的文本编辑器。它以高效、灵活和高度可定制著称，广泛用于编写程序代码、配置文件编辑等场景。

本文详细的介绍Vim文本编辑器的常用功能和实用技巧。Vim以其高效、灵活和可定制性强的特点，成为程序员和系统管理员的首选编辑工具。文章首先讲解了Vim的基本操作，如普通模式与插入模式的切换、文本搜索与替换、多窗口编辑等常见功能。随后，通过具体实例，详细介绍了提升编辑效率的多个技巧，包括快速导航、块选择与操作、搜索与替换以及个性化配置等内容。通过掌握这些技巧，用户可以在实际工作中更加高效地使用Vim进行文本编辑，提升工作效率，优化操作体验。

详细内容请参考下文。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/4b46af3212f7435ab507d436beabc70b~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=G29AwdvgBiR1uCeBz6qkUPAKL4g%3D)



**一、登录Linux系统**

**1.访问Linux系统**

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/4ddaa5e79e6045f5b62a463ef34daa17~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=zRFwCitGBZwWfCqh9iy113p0%2F7o%3D)



**2.查看Vim版本信息**

![img](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/8a63f98e69444abba83873db85679764~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=N3sIYcPbjOYi9P2g6mqrL5MSWXs%3D)



**二、Vim简介**

**1.** **Vim是什么**

说明：Vim（Vi IMproved）是一个功能强大的文本编辑器，是Vi编辑器的增强版本。它提供了丰富的功能和灵活的配置选项，适用于各种文本编辑任务，尤其在编程和系统管理中非常流行。

Vim的设计理念是让用户能够高效地编辑文本，支持多种编程语言和文件格式。它具有强大的搜索、替换、宏录制和插件系统，使得用户可以根据自己的需求进行高度定制。

**2.** **Vim的特点**

说明：Vim的使用需要一定的学习时间，但一旦掌握，其高效的编辑方式和丰富的功能将大大提高文本编辑的效率。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/9668058212fc41ffa22b2baaf0d5c2bd~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=jnH4hCsSfF32Dky71L4l91c6IKU%3D)



**三、Vim基本操作**

**1.** **Vim的三种模式**

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/47e31fd67a574a068c97e0dc6523578a~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=FoMDpn5ea8obbiussgx4yn%2BDR7Q%3D)



**2.** **Vim的操作技巧**

**(1)进入和退出Vim**

- 进入Vim：在终端输入“vim filename”。
- 退出Vim：在普通模式下，输入“:q”退出，输入“:wq”保存并退出。 如果有未保存的更改，可以使用“:q!”强制退出而不保存。

**(2)基础操作**

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/5272fdf4afaf4e6da5355dbb60372457~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=ALZgKoUCA1LnC0aDpnuicorHFfU%3D)



**(3)文本编辑**

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/2186b6289a184d2aaa06de7ce6c1e609~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=K8Y5Hrf8J1D%2BwEgFkRiskwaLttQ%3D)



**(4)选择与操作**

说明：Vim允许用户以可视模式选择文本块进行操作。进入可视模式的方法是按v（逐字符选择）、V（逐行选择）或者Ctrl + v（列选择）。选定文本后，你可以进行复制、剪切、替换等操作。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/f2c1ecd17d0c4458ae491e58becea03f~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=Y2bdCojG07iXB8TgC6QZQu6E1Nc%3D)



**(5)搜索和替换**

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/6c8c334088734334a62a489742668b88~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=LRwX%2BdKZH6HEyD4WE9oBryBkCz4%3D)



**(6)其他命令**

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/be147484afcf4e8ca40757b399480900~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=VwAi7NlGiIglfN4U9CBDX6KzJPU%3D)



操作实例：

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/965fd6ed40b6494cacc6a4a926681d8e~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=c%2FRghQvbj9bs7bIKEKvS8FKJWU4%3D)



输入:set number

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/fff36a00af31482daf5d5414b3d12216~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=nTNuCGRauD%2BZyOUbzPATMSqsoT8%3D)



**(7)自定义Vim**

说明：一般情况下，用户配置文件位于主目录 ~/.vimrc，通过执行 vim ~/.vimrc 命令即可对此配置文件进行合理修改。通常情况下，Vim 用户配置文件需要自己手动创建，创建后可以在其中添加自定义设置。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/ba26ee0c820e47db98d69ec82de855db~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=08JqBGs28vfQSXN2kJ%2FUDZ2wvQM%3D)



使用实例：

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/0eed777103004fdba850c4f832cc0245~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756684903&x-signature=aSVPByrELTTysBb4Paj8xpujPQE%3D)



