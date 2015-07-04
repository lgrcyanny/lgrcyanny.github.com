title: 谈谈开源软件，谈谈质量
tags:
  - Learning
  - Methodology
id: 278
categories:
  - Life
date: 2013-10-06 11:25:17
---

开源软件的基本概念最早可追溯到1955年IBM为了交换编程资料、深入研究IBM操作系统而开发的“IBM用户组分享”。在几十年后，1997年Eric Raymond发表了《大教堂与市集》——开源软件运动的圣经，以大教堂和市集这生动的比喻，阐述了商业封闭软件和自由软件的开发模式，并以Linux和Fetchmail为案例讨论了开源软件的优势。至今，20多年过去了，开源软件取得了不朽的成就，像Linux，Java，Eclipse，Android，Apache，Hadoop，Git，Nginx，Python，Node.js等，都是优秀的开源软件。

### 开源软件vs自由软件

##### 开源软件

开源软件的定义，是由Bruce Perens(Debian创始人之一)于1997年基于Debian Free Software Guidelines提出，开源软件即公开源代码的软件，允许用户使用，修改，拷贝，合并，出版发行，再散布，贩售软件及软件副本，同时开源软件允许版权持有人在软件协议的规定之下保留一部分权利。

open source describes a broad general type of software license that makes source code available to the general public with relaxed or non-existent copyright restrictions.[more...]

1998年Eric.S.Raymond和Perens成立了OSI(Open Source Initiative), OSI规定开源软件需要遵守OSI支持的许可证，例如Apache License, BSD license, GNU General Public License, GNU Lesser General Public License, MIT License, Eclipse Public License和Mozilla Public License，如果软件源代码本身开源，但是限制用户的修改和再发行等权利，或者程序源码不可读、质量低造成用户难以自由使用，也只能是“Source Available Software”，不是开源软件。

##### 自由软件

相对于开源软件，我们常提到自由软件，大多数开源软件也是自由软件，但二者不能混谈。自由软件, 是1983年由美国麻神理工人工智能实验室研究Richard Stallman提出，并于1984年发起了自由软件运动，建立了自由软件基金会(FSS)，启动了GNU工程。他提出了Copyleft的思想，并为Copyleft提出了GPL许可证，GPL是该运动的精髓。自由软件的定义有FSS在1986年正式提出：
software is free software if people who receive a copy of the software have the following four freedoms.

Freedom 0: The freedom to run the program for any purpose.

Freedom 1: The freedom to study how the program works, and change it to make it do what you wish.

Freedom 2: The freedom to redistribute copies so you can help your neighbor.

Freedom 3: The freedom to improve the program, and release your improvements (and modified versions in general) to the public, so that the whole community benefits.

Freedoms 1 and 3 require source code to be available because studying and modifying software without its source code can range from highly impractical to nearly impossible.

##### 开源软件和自由软件区别

如果硬要区别开源软件和自由软件，可以说二者的出发点不同，用Richard Stallman的话说"Open source is a development methodology; free software is a social movement". 

开源软件出发点是程序本身，旨在通过开源，让更多的人合作开发，Linus定律指出"Given enough eyeballs all bugs are shallow.",开源致力于提高程序的质量，是一种集市的开发模式.

自由软件可以看做一种哲学和道德观念，强调用户的自由，如果一个软件卖给一个用户的只是二进制代码，而没有源代码，妨碍了用户自由使用、学习的权利，这就是不道德的。Richard Stallman认为软件不应该用来剥削和压榨利润，应该给用户自由的权利。自由软件一定是开源的，自由软件可以看做是开源软件的子集，开源软件中存在一些软件是限制用户再发售软件的权利，这就不是自由的。但大多数软件是自由开源软件FOSS。

The term “open source” software is used by some people to mean more or less the same category as free software. It is not exactly the same class of software: they accept some licenses that we consider too restrictive, and there are free software licenses they have not accepted. However, the differences in extension of the category are small: nearly all free software is open source, and nearly all open source software is free.

-----[Free Software Foundation](http://www.gnu.org/philosophy/categories.html) 

自由软件常用的许可证是GPL和BSD，现在的更多的FOSS接受MIT或者BSD的许可证，而不太接受自由软件协议GPL，因为在GPL下，再次开发的软件必须是GPL的，不能作为商业用途，代码的传染性太高，也算是一种对自由的限制。

### 开源软件的优势

1.较低的维护成本，快速的Bug修复

正如Linus定律所述: “如果有足够多的眼睛，所有的错误都是浅显的。”更多的开发者，则可以更快的修复Bug。程序员如果找到Bug，不比像商业软件中那样，先上报、再审批、再决定Bug修复方法，再修复，效率较低，而开源社区的程序员们则可以快速响应修复问题。并且，开源软件的开发者们都是自主自愿的做贡献，软件的维护成本基本为零。

2\. 自由和透明性

开源软件让开发者可以自由地使用和学习源代码，可以按自己的需求修改源代码，可以学习别人的代码，提高自己的技能。同时也可以了解软件的运作机制，修复bug，为开源软件做出贡献，提高自己在科技界的知名度。 这是开发者的快乐所在。

3\. 为公司打开市场

IBM公司在开源软件Eclipse上投入上亿美元，开发了这个高质量的Java IDE，打败了竞争对手，推广了自己的产品，赢得了市场；Joyent公司发布开源的Node.js, 一个基于Chrome V8引擎的开发web应用的开发平台，曾一度成为GitHub上的Top Star，并受MicroSoft, Linkedln，eBay等大公司的亲睐，相信Joyent这家云服务提供商也拓宽了自己的市场。在软件有了市场后，便可以通过对软件的互补物品，如技术支持，存储空间，广告等方面获得利润。

4\. 吸引更多优秀的工程师

一个公司进行一个开源项目，可以吸引到更多的人才。很多工程师，喜欢为开源社区工作，提高自己在科技界的声望，帮助他们找到更好的工作。而如果有公司支持开源项目，他们也乐意去那里工作。

### 开源软件的质量不太理想

开源软件承诺并鼓励开发更高质量，更高可靠性，更加灵活，更低成本的软件，让用户得到更多的自由，而我们也看到像Linux，GNU Project, Apache Project, Python, Chromium, Mozilla, Android都是高质量的开源软件，然而开源软件并不保证一定高质量，在[ohloh(开源和自由软件的在线字典)](http://www.ohloh.net/ "ohloh")上的60多万的开源软件的质量没有那么理想，高质量的开源软件是榜样，但不能把开源和高质量等同。
Robert在《软件工程的事实与谬误中》给出了质量的定义，即7个属性的集合：可靠性，可移植性，效率，可用性（人类工程学），可测试性，易理解性和可修改性。
开源软件在质量方面的劣势如下：

1\. 可用性不理想

商业软件，如Windows，Adobe Photoshop，Office，Dropbox，Skype，iTune等，都有对应的开源软件，如Ubuntu，GIMP，Open Office，Cabos，CuteCom，Songbird，同时也是免费的。开源软件的开发者们缺乏在可用性，HCI方面的设计经验，常常会去模仿工业界的商用软件的设计，缺乏自主创新，这也就解释了几乎每一个商业软件的都有一个开源软件与之对应。模仿的结果是可用性和用户满意度不高，对于不懂技术的普通用户，如果经济允许，购买商用软件会比用开源带来更多的方便。

2\. "早发布，常发布"造成缺乏成熟的设计

开源社区，有优秀的程序员，但少有优秀的设计师。开源软件让开发者合作开发，其本质是大量的开发者为软件免费修复bug。开源软件还没有设计好，就草率的发布，势必会降低软件的质量，软件最终会失控。而很多时候，软件设计的重要性等同甚至大于代码的重要性，如HCI设计，需要专业的设计师。而开发者们都喜欢去实现算法，编写代码，在可用性上只按自己的喜好进行设计，造成在细节方面做得不如商业软件。

3\. 文档质量不高

很多时候拿到一份源代码，看文档比看源码更有效率，文档是交流的利器。然而，开源软件的文档质量并没有商业软件那么理想。商业软件有好的技术文档和用户手册，但开源软件的开发者宁可写代码，也不太喜欢写文档。我曾经参与过Joomla项目，Joomla的文档就让人很头疼。

4\. 软件变得越来越复杂而难以控制

太多的人参与，太多的意见，开发者们按自己的需求，自己对"好软件"的定义开发，让软件越来越复杂，集成和版本控制的成本增加，这些都源于对用户需求没有一个清晰的定义，不满足用户的需求，或者过多冗余的功能，会造成软件的质量下降。

5\. 开发人员管理困难

好的软件往往需要一个或多个优秀的开发者全职在一起工作，一起开发、维护和改善软件。当开发者遍布世界各地，便很难保证开发者会全职为软件做贡献。而很多优秀的开源软件是大公司支持的，软件的贡献者同时也是公司的全职员工，这也是这些开源软件可以高质量的一个原因。

### 开源软件和商业软件之间的平衡

软件开发没有能带来数量级增长的银弹，开源和商业软件各有优势，如何选择是需要仔细考虑的。

1\. 根据用户的需求来决定用开源软件或者商业软件

开源软件，开放源代码，开发者对此更感兴趣，如果应用是面向开发者的软件，对可用性和HCI要求不高，如编程语言（Python），服务器，IDE，平台软件，可以考虑用开源软件，让大量的开发者合作开发，可以提高软件的质量。

如果用户是不关心技术的普通用户，且如果有对软件保密性、安全性、高可靠性等方面的需求，采用商业软件的开发方法更好，例如金融理财软件，邮件系统，铁路系统等。
如果用户对软件的交付日期有要求，不要用开源软件，开源很难控制开发进度。

2\. 如果采用开源软件，当心GPL

很多开源软件受GPL的保护，而如果你需要二次开发的开源软件，在GPL下，则你开发的软件也必须在GPL下，这会损害你的专利权利，并且软件不能出售。

3\. 考虑个人的需求和公司的需求

如果对于开发者个人，需要提高自己在软件界的声望，想锻炼自己的能力，让CV更加好看，多参与和贡献开源软件是一个好方法。

如果一个公司，想扩大自己的市场影响力，想以开源软件的方法获得商业利益，则可以采用开源软件。

但如果个人或者公司想创新，实现创新的idea，并保有自己的专利权利，打算以出售软件来赚钱，那么就要采用商业软件的开发方法。

总之，有了开源软件，我们就多了一种软件的开发方法，开源软件为软件的世界带来了很多生机，像GitHub现在全球最大的开源社区，每天都有世界各地的开发者合作交流。商业软件还是开源软件，我们都要从自己的需求出发来决定。

### 一些参考资源

[When Free Software Isn't (Practically) Better](http://www.gnu.org/philosophy/when_free_software_isnt_practically_better.html "When Free Software Isn")

[Why free software has poor usability, and how to improve it](http://www.mpt.net.nz/2012/06/why-free-software-has-poor-usability/ "why-free-software-has-poor-usability")

[The Problems of Open Source](http://www.lambdassociates.org/blog/the_problems_of_open_source.htm "the_problems_of_open_source.htm")

[Corporate Open Source Considerations](http://www.gilyehuda.com/2011/01/12/open-source-considerations/ "open-source-considerations")

[Open Source Sucks](http://www.hackerfactor.com/blog/index.php?/archives/415-Open-Source-Sucks.html "Open-Source-Sucks")

[Open Source Rocks](http://www.hackerfactor.com/blog/index.php?/archives/416-Open-Source-Rocks.html "Open-Source-Rocks")

[The Top 50 Proprietary Programs that Drive You Crazy — and Their Open Source Alternatives](http://whdb.com/blog/2008/the-top-50-proprietary-programs-that-drive-you-crazy-and-their-open-source-alternatives/ "the-top-50-proprietary-programs-that-drive-you-crazy-and-their-open-source-alternatives/")

[Open Source Software: The Hidden Cost of Free](http://www.forbes.com/sites/rajsabhlok/2013/07/18/open-source-software-the-hidden-cost-of-free/ "open-source-software-the-hidden-cost-of-free")