#UTF-8无处不在宣言

##1 这份文档的目的

我们的目标是促进 UTF-8 编码的使用与支持，让人们确信 UTF-8 编码应该是在内存或磁盘中存储文本字符串，用于沟通和所有其他用途的默认编码选择。我们相信我们的方案提升了性能，降低了软件的复杂度并且避免了很多 Unicode 相关的错误。我们相信所有其它 Unicode（或文本）的编码适用于优化极少的边界情况，主流用户应该避免使用。

特别地，我们认为非常流行的 UTF-16 编码（在 Windows 世界被错用做宽字符（widechar）和 Unicode 的同义词）不应该用在库 APIs 中，除非用于处理文本的特殊的库如 ICU。

本文档建议在 Windows 应用程序中也使用 UTF-8 编码存储字符串，尽管由于历史原因和原生API级别 UTF-8 支持的缺乏这个标准在这个领域并不流行。我们相信即使在这个平台上，后面的一些观点胜过原生支持的缺乏。并且，我们建议永远忘记内码表（ANSI codepages）和它们的用途。在文本字符串中混用任意种语言是属于用户的权利。

在工业上，很多本地化相关的 bugs 源于程序员缺乏 Unicode 编码知识。然而，我们认为，如果一个应用不是专业用于文本，它的架构应该能够让程序无需考虑编码问题。举例来说，一个文件复制工具不应该为了支持非英语文件名而写的不同。在这份宣言里，我们还将告诉一个程序员如果他不想陷入 Unicode 的复杂之中并且不关心字符串的内部，他应该怎么做。

此外，在文本处理情景中，我们建议不应该将计数或者遍历 Unicode 码点视为特别重要的任务。许多开发者错误地将码点视为 ASCII 字符的继承。这导致了像 Python 字符串 O(1) 码点获取。然而，事实上 Unicode 非常复杂并且没有像 Unicode 字符这样的通用定义。相对与 Uncoide 字符族，码元或者甚至语言中的单词，我们没有特殊的原因去更青睐 Unicode 码点。另一方面，将 UTF-8 码元视为一个文本的基本单元对于很多任务来说特别有用，如解析常用的字符数据类型。这源于这种编码的特性。字位，码元，码点和其他相关的 Unicode 相关的术语在部分5有解释。编码字符串的操作在部分7有讨论。

##2 背景

1988年，Joseph D. Becker 发布了[第一个 Unicode 草案提议](http://unicode.org/history/unicode88.pdf)。他的设计的根据是一个天真的假设:每个字符 16 位就足够。在1991年，Unicode 标准的第一个版本发布，码点被限制在16位。在接下来的几年中，许多系统添加了对 Unicode 的支持并且转换到了 UCS-2 编码。对于像Qt框架，(1992)，Windows NT 3.1(1993)和 Java (1995)等当时的新技术，它是非常有吸引力的。

然而，很快就发现每个字符16位对于 Unicode 是不够的。在1996年，UTF-16编码被创造出来，从而使现存的系统能够处理非16位的字符。这使得最初选用16位字符编码，也就是固定宽度编码的理论没有了依据。现在，Unicode 包含超过109449个字符，其中大约74500个属于中日韩（CJK）统一表意文字。

![image](https://github.com/mxui/UTF-8Everywhere/blob/master/nagoya-museum.jpg)

_名古屋科学博物馆           Vadim Zlotnik摄。_

微软经常错误地将 Unicode 和宽字符 (widechar) 作为了 UCS-2 和 UTF-16 的同义词。而且，因为 UTF-8 不能被设置为窄字符串的 WinAPI 的编码，人们必须定义 UNICODE 编译代码。Windows C++ 程序员被教育 Unicode 必须和宽字符（甚至是使用与编译器设置相关的TCHARs，允许程序员不支持全部的 Unicode 码点）一起使用。作为这些混乱的结果，许多 Windows 程序员在关于如何正确处理文本非常疑惑。

与此同时，在 Linux 和 Web 世界里，有一个默认的共识：UTF-8 是地球上对 Unicode 最好的编码。尽管相对于其它的文本，它对英语，包括计算机语言（例如 C++，HTML，XML等）提供了更短的表示，在常用的字符集上，它也很少比 UTF-16 编码效率低。

##3 事实

* 在 UTF-8 和 UTF-16 编码中，码点都是最多占据 4 个字节。

* UTF-8 是端性中立的。UTF-16 有两种格式：UTF-16LE 和 UTF-16BE（分别对应不同的字节序）。这里我们将他们统称为UTF-16。

* 宽字符（widechar）在一些平台上2个字节大小，在其他平台上4个字节。

* 当按字典序排序的时候，UTF-8 和 UTF-32 顺序相同，UTF-16则不同。

* UTF-8 编码处理在英语字母和其它 ASCII 字符上效率更高（每个字符一个字节），而 UTF-16 编码则在处理几个亚洲字符集上效率更好（每个字符两个字节而不是 UTF-8编码中的3个）。在 Web 世界中，英语的 HTML/XML 标签和各种文本混在一起，这使得 UTF-8 成为了最受欢迎的选择。西里尔字母，希伯来语和几个其它流行的 Unicode 字符表块在 UTF-16 和 UTF-8 中都是2个字节。

* UTF-16 经常被误用做固定宽度的编码，即使在 Windows 原生应用中：在普通文本编辑控制中（直到 Vista）,删除一个在 UTF-16 中占据 4 个字节的字符，要使用两个退格键。在 Windows 7中，控制台将该字符显示为两个无效字符，无论使用什么字体。

* 许多 Windows 第三方库不支持 Unicode：它们使用窄字符串参数并且将它们传递给 ANSI API。即使文件名也是这样。总的来说，这样是行不通的，因为字符串不可能使用一个 ANSI 页码表示完全（如果它包含来自不同 Unicode 区块的字符）。在 Windows 上解决文件名问题的方案是添加一个 8.3 路径到这个文件上并添加到该类库上。如果这个库需要建立一个不存在的文件，这将不可行。如果路径非常长并且8.3格式比 MAX_PATH 长，这也将不可行。如果在操作系统设置中短名生成不可行，这仍将不可行。

* 在C++中，除了使用 UTF-8 编码，没有方法能够从 std::exception::what() 返回 Unicode。除了使用 UTF-8，没有别的办法使 Unicode 支持 localeconv。

* UTF-16 在当今仍然流行，即使在 Windows 世界之外。Qt，Java，C#，Python(在 CPython 3.3版本参考实现之前，[参考](http://utf8everywhere.org/#faq.python))和 ICU —— 他们都是用 UTF-16 作为内部字符串表示。

* 在 Linux 世界中，窄字符串在各个地方被默认为 UTF-8 编码。举例来说，一个文件复制工具不需要在意编码问题。一旦ASCII字符串作为文件参数测试通过，任何其他语言的文件名都会工作正常，as arguments are treated as cookies(怎么翻译？).为了支持其他语言，复制工具的代码一点也不需要改变。fopen()无缝的接收 Unicode 字符，argv 也同样如此。

##4 透明数据争论

让我们回到文件复制工具。在 UNIX 世界中，窄字符串几乎到处被默认为 UTF-8 编码。因此，文件复制工具的作者不需要考虑 Unicode。一旦 ASCII 字符串作为文件名参数测试通过，任何其他语言的文件名都会工作正常，as arguments are treated as cookies. 复制工具的代码一点也不需要改变就可以支持其他语言。fopen() 无缝的接收 Unicode 字符，argv 也同样如此。

现在，让我们看一看在基于 UTF-16 架构的微软 Windows 上如何实现。在这上面制作一个能够接受几种不同的 Unicode 块字符文件名参数的文件复制工具需要高深的技巧。首先，这个应用必须编译为 Unicode 感知的。在这个例子中，它不能有带有标准C参数的 Main() 函数。它将接受 UTF-16 编码的 argv。为了使一个核心部分使用用窄字符文本的 Windows 应用程序接收 Unicode 字符，它必须深层次的重构并且认真处理每一个字符串变量。

和 MSVC 一同发布的标准库没有很好的支持 Unicode。它直接将窄字符串转发到系统的 ANSI API。没有办法重写这些。改变 std::locale 也无效。在 MSVC 上使用 C++ 的标准特性无法打开一个使用 Unicode 字符名称的文件。打开一个文件的标准方法是：

                       `std::fstream fout("abc.txt")`

解决这个问题的方式是使用微软自身接受宽字符参数的hack，但这不是标准的扩展。

在 Windows 上，_HKLM\SYSTEM\CurrentControlSet\Control\Nls\CodePage\ACP_ 注册表键值能够使接受非 ASCII 字符，但是这些字符只能来自单个 ANSI 字符页。在 Windows 上，一个未被实现的值 65001 将能够解决 cookie 问题。
如果微软实现对这个 ACP 值的支持，这将有助于 UTF-8 编码在 Windows 平台上更广泛的应用。

对于 Windows 程序员和多平台库的开发商，我们会在 [Windows平台如何处理文本]() 部分深入探讨我们为了更好支持Unicode 的处理文本字符串和重构程序的方法。

##5 Glyphs, graphemes 和其他 Unicode 属性  

这里有一个带有我们评论的按照 Unicode 标准的关于字符，码点，码元和象形聚簇定义的摘录。你可以参考标准的相关部分获取更多详细的描述。

**码点 (code point)**

Unicode 编码空间的任何数值。举例：U+3243F。

**码元 (code unit)**
   
能够代表一个编码文本单元的最小位组合。举例，UTF-8，UTF-16 和 UTF-32 分别使用 8位，16位和 32 位的码元。上面提到的码点将
会被编码为 ‘f0 b2 90 bf’使用UTF-8，'d889 dc3f' 使用 UTF-16， '0003243f' 使用 UTF-32。注意这些仅仅是位组的次序；它们如何
被存储进一步取决于相关编码的端性。所以，当存储上面的 UTF-16 码元在一个 octet 的媒介上，他们将会被转化为 'd8 89 dc 3f'
在 UTF-16BE 编码中，在 UTF-16LE中则是 '89 d8 3f dc' 。

**抽象字符（Abstract character）**

用来组织，控制和表示文本数据的一组信息单元[§3.4, D7]。在 §3.1 中标准进一步说明：
   
>对于 Unicode 标准，[...] 这个字符集本质上是开放的。因为Unicode是一个通用的编码，任何被编码过的抽象字符都将是被编码的候选，
>不管这个字符现在是否为人们所知。

这个定义真的很抽象。任何人们能够想到可以作为字符都是一个抽象字符。举个例子，![](https://github.com/Mjinrui/UTF-8Everywhere/blob/master/glyph-ungwe.png) tengwar字母 ungwe 是一个抽象字符，虽然它还不能在 Unicode 中表示。
   
**编码字符（Encoded character，Coded character）**

一个码点和一个抽象字符的映射。
举例来说，U+1F428 是一个已编码的字符代表抽象字符 🐨 KOALA。
   
这种映射既不是完整的，也不是单射，也不是满射：
  
* 代表，非字符和未分配的码点不对应抽象字符。
* 一些抽象字符可以使用不同的码点编码；
  U+03A9 希腊大写字母 OMEGA 和 U+2126 欧姆符号 都对应相同的抽象字符 'π' ，并且必须被同样的看待。
* 一些抽象字符不能被一个单一码点编码。它们用一序列已编码字符表示。举例来说，表示抽象字符
  ю́ cyrillic small letter yu with acute 的唯一方法是用序列 U+044ECYRILLIC SMALL LETTER U
  紧跟 U+0301 COMBINING ACUTE ACCENT.
 
还有，对于一些抽象字符，除了使用单码点的已编码字符的形式，还存在着使用多码点的表示方法。抽象
字符 ǵ 可以用单一码点 U+01F5 LATIN SMALL LETTER G WITH ACUTE 编码，或者被<U+0067LATIN SMALL LETTER G, 
U+0301 COMBINING ACUTE ACCENT> 序列编码。

**用户感知字符（User-perceived character）**
  
任何终端用户看为一个字符的字符。这个概念是依赖于语言的。举例来说，'ch' 在英语和拉丁语中是两个字母，但是
在捷克语和 Slovak 中被看做一个字母。
   
**字形族（Grapheme cluster）**

一个应该保持在一起的已编码字符序列。字形族在单独语言的角度看接近用户感知字符。他们被
用来做光标移动和选择。

字符可能意味着上面的任意定义。Unicode 标准将它作为已编码字符的同义词[§3.4]。当一些编程语言或者库文件说到‘字符’，这总是意味着一个码元。当一个终端用户被问到一个字符串中字符的数量的时候，她将计数用户察觉的字符。根据一个程序员的的 Unicode 熟悉程度，他会将码元，码点或 grapheme 聚簇数做字符。例如，[推特如何计算字符数量](https://dev.twitter.com/docs/counting-characters)。我们认为，对字符串 '🐨’'，一个求字符串长度函数不一样一定返回 1 才是兼容 Unicode 的。 

##6 亚洲文本：UTF-8 vs UTF16

大多数的 Unicode 码点使用 UTF-8 和 UTF-16 编码占据相同的字节数。这包括俄罗斯文，希伯来文，希腊文。并且所有的 non-BMP （非基本面）码点在两种编码中都占有2个或4个字节。尽管一些亚洲字符使用 UTF-8 占用更多空间，使用 UTF-16 编码会在拉丁字母，标点符号和其余的 ASCII 字符上占用更多的空间。亚洲程序员们怎么能够不反对放弃每个字符节省他们 50% 内存的的 UTF-16编码呢?

事实上，只有在人为构建的仅包含 U+0800 到 U+FFFF 范围内字符的例子中才是这样。然而，电脑和电脑间的文本接口比文本的其他应用要广泛。这包含 XML，HTTP，系统路径和配置文件——他们几乎都使用独家的 ASCII 字符，并且事实上在相关的亚洲国家 UTF-8 使用的同样多。
 
对于中文书的专门存储，UTF-16 依旧可以被使用来作为不错的优化。一旦文本从这样的存储中取出来，它应该被转换为和世界相兼容的标准。在两种情况下，如果存储是昂贵的，无损压缩总是会被使用。在这样的情况下，UTF-8 和 UTF-16 将占据大致差不多的空间。进一步，“在语言中，一个象形文字比拉丁字符传递更多的信息，所以它占据更多的空间也很合理”（Tronic,[UTF-16 harmful](http://programmers.stackexchange.com/questions/102205/should-utf-16-be-considered-harmful/102211#102211)）。

这里有一个简单实验的结果。一些网页（日语文章，2012年1月1日采集于日语维基百科）的 HTML 源文件占据的空间在第一纵列。第二纵列显示的是去除标记后的这些文本的结果，就是选择全部，复制，粘贴到纯文本文件中。

	              |HTML Source (Δ UTF-8)      |Dense text (Δ UTF-8)
|--------------------|---------------------------|--------------------
| UTF-8	       |767 KB  (0%)	              |222 KB  (0%)
| UTF-16	       |1186 KB (+55%)	       |176 KB  (−21%)
| UTF-8 zipped	|179 KB  (−77%)	       |83 KB   (−63%)
| UTF-16LE zipped	|192 KB  (−75%)	       |76 KB   (−66%)
| UTF-16BE zipped	|194 KB  (−75%)	       |77 KB   (−65%)

可以看出，UTF-16 在实际数据上占据的空间比 UTF-8 多 50%，对于纯亚洲文字仅仅节约 20% 的空间并且在用通用压缩算法处理后很难有竞争力。

##7 编码字符串的文本操作
流行的基于文本的数据格式（如CSV, XML, HTML, JSON, RTF和计算机程序的源码）经常包含 ASCII 字符作为结构控制元素并且包含ASCII和非ASCII数据字符串。处理ASCII字符码点长度比其他码点长度短的变长编码似乎是一个困难的任务，因为字符串内的编码字符的界限不是立即可知的。这使得软件架构选择 UCS-4 定长编码(例如，[Python v3.3](http://utf8everywhere.org/#faq.python))。事实上，这既是不必要的，也并没有解决我们知道的任何实际问题。

自UTF-8设计之初，UTF-8 保证一个 ASCII 字符值或者子串永远不会匹配到一个多字节编码字符的一部分。UTF-16也是这样。在两种编码中，多部分编码码点的码元会将 MSB 设置为1。

'<' 符号标志着一个 HTML 标签的开始，或者一个UTF-8 编码的 SQL 语句中防止 SQL 注入，正如你讲作为一个全英语的文本 ASCII 字符串。编码保证这个能够运行。特别地，每一个非 ASCII 字符都作为字节序列编码为UTF-8的，序列中的每一个都有一个大于 127 的值。这使得原有的算法不可能发生碰撞 --- 简单，快速和优雅，并且不需要在意已编码字符边界。 

并且，在从一个 UTF-8 编码字符串中查找一个非ASCII，UTF-8 编码的子串时你可以将它看做一个纯字节数组，不需要关心码点的边界。这归功于 UTF-8 编码的另一个设计特性----一个码点前面的字节永远不能与任何其他码点的后面的字节之一有相同的值。

##8 在计算字符上更深的谜题

正如我们已经说明的，在一个 Unicode 字符串中计数，分割，索引和其他遍历码点的操作应该被视为经常和重要的操作。在这部分，我们将认真地重看这部分。

1. 使用 UTF-16，计算字符数可以在一个常量时间内完成。

这是那些认为 UTF-16 是一个固定宽度编码的人的一个共有错误。不是这样的，事实上 UTF-16 是做一个变长编码。如果你依然否认非BMP 字符的存在，参考[这个问答](http://utf8everywhere.org/#faq.almostfw)。

2. 使用 UTF-32，计算字符数量可以在一个常量时间内完成。

这取决于被误用的词语 ‘字符’ 的含义。在 UTF-32 中，我们是可以在常数时间内计数码元和码点。然而，码点
并不一定对应用户可觉的字符。即使在 Unicode 形式中一些码点对应编码字符，一些对应非字符。

**给已编码字符或者码点计数是重要的**

码点的重要性经常被夸大。这是由于对仅仅反映人类语言复杂性的 Unicode 复杂度的误解。很容易说出在 'Abracadabra' 中有多少个字符，但是对于下面的字符串却并不是那么简单：

                                Приве́т नमस्ते שָׁלוֹם
                                
上面的字符串包含 22(!) 码点，但仅仅有 16 个象形族。所以，‘Abracadabra’ 包含 11 个码点，上面的字符串包含 22 个码点，如果转换为 [NFC](http://unicode.org/reports/tr15/) 则进一步仅有 20 个码点。然而，码点的数量和几乎与任何软件工程问题无关，唯一的例外是将字符串转为 UTF-32。举个例子：

* 对于光标移动，文本选择，象形族将会被使用。
* 对于在输入领域，文本格式，协议和数据库上限制字符串长度的情况，长度是根据提前指定的编码的码元测算出来的。原因是任何长度限制是来源于在低层次给字符串分配的固定量的内存，在内存中，磁盘中或者在一个特定的数据结构中。
* 字符串在屏幕上显示的大小和字符串中码点的数量无关。为了解这些，不得不通过渲染引擎。码点不占据一列即使在终端和等宽字体中。POSIX 考虑到了这点。

4.在 NFC 中，每个码点对应一个用户可查的字符

不，因为可以在 Unicode 中表示的用户可见字符的数量是无限的。即使在实践中，大多数字符并没有一个完全组成的形式。举个例子，上面例子中的包含三个真实语言的三个真实词语的 NFD 字符串将在 NFC 中包含 20 个码点。这远比它包含的 16 个可见字符多。

5.length() 字符串操作一定计算用户可见字符或者已编码字符的数量。如果不是，它就没能正确地支持 Unicode。

程序库和编程语言对 Unicode 的支持经常通过返回字符串长度操作的值来判断。根据对 Unicode 支持的测试，大多数流行的语言，例如 C#,Java 甚至连 ICU 都不支持 Unicode。举个例子，一个字符串 ‘🐨’的长度在UTF-16 作为内部字符串表示的地方经常被报告为2，在内部使用 UTF-8 的语言中则是 4。误解的来源是这些语言的说明中使用字符表示一个码元，尽管程序员希望它是其它的东西。

因此，这些 API 返回的码元数实际上非常重要。当写一个 UTF-8 字符串到一个文件，最重要的是字节数的长度。计数任何类型的字符，并不是非常有用。

##9 我们的结论

UTF-16 的两个方面都是最糟糕的，长度可变和太宽。它因为历史的原因而存在，造成了许多的迷惑。我们希望它的使用进一步减少。

可移植性，跨平台协作和简单性这些都比和现有平台的 APIs 相容更重要。所以，最好的方案是在 Windows 上每个地方都使用 UTF-8 窄字符串并且在调用接收字符串的 APIs 前将它们来回转换。当处理接收字符串的系统 APIs 时，性能几乎不与此相关，但是在各个地方使用相同的编码有巨大的好处，我们没有足够的理由不去这么做。

说到性能，机器经常使用字符串进行通信（如HTTTP头，XML）。许多人将此视为错误，虽然这几乎总是通过英文完成，在这个方面给了 UTF-8 更多的优势。不同的字符串使用不同的编码增加了复杂度和因此造成的bugs.

尤其，我们认为给 C++ 增加 wchar_t 类型是一个错误，C++11添加的 Unicode 补充也一样。虽然这样，最需要实现的是 基本执行字符集能够存储任何 Unicode 数据。然后，每个 std::string 或者 char* 参数都必须是 Unicode 兼容的。 ‘只要接受文本，就是应该是 Unicode 兼容的'——使用 UTF-8，这点很容易做到。

这个标准的部分有许多设计瑕疵。这把包括 std::numpunct，std::moneypunct 和 std::ctype都不支持变长编码字符 （非ASCII的 UTF-8和 非BMP的 UTF-16）。他们必须被修正：

* decimal_point() 和 thousands_sep() 应该返回一个字符串而不是一个码元。这就是 C locales 通过 localeconv函数支持这个，尽管并不可定制）

* toupper() 和 tolower() 在和码元（code units）相关的时候也不应该被使用，因为它不支持在 Unicode 下工作。举例来说，拉丁连体字符 ffl 必须被转换成 FFL，德语 ß 必须被转化为 SS（有一个大写形式的ẞ，但是转化规则遵循传统的方式）。除此之外，一些语言（例如希腊语）有一些小写字母的最终特殊形式，这种情况下转换必须清楚正确地进行转换的位置。

##10 在 Windows 上如何处理文本

这部分是为了多平台库开发和 Windows 平台编程。Windows 平台的问题是它不支持与 Unicode 兼容的窄字符串系统 APIs。将 Unicode 字符串传给 Windows API 的唯一方式是通过转化成 UTF-16（即宽字符）。 

注意我们的说明和微软的关于 Unicode 转换的原始说明显著不同。我们基于实施宽字符转换的方法尽可能与 API 调用相近，并且不处理宽字符数据。在上一部分我们解释了这将导致更好的性能，稳定性，代码简介和其他软件更好的集成。

* 不要在任何地方使用 wchar_t 或 std::wstring，除非在和接受 UTF-16 的APIs的临近点。
* 不要在任何地方使用 _T("") 或 L""修辞，除非用在接受 UTF-16 的APIs的参数上。
* 不要使用对 UNICODE 常量敏感的类型，函数或者他们的衍生物，例如 LPTSTR 或者 CreateWindow() 或 _T 宏。相反，使用 LPWSTR, CreateWindowW() 和相近 L"" 字符量。
* 为了避免传递给 ANSI WinAPI 的 UTF-8 字符串被悄悄地编译，Unicode 和 _UNICODE 仍总是要被定义。
* 程序中任何地点出现的 std::strings 和 char* 都被看做 UTF-8 。
* 如果你有幸写 C++，下面的 narrow()/widen() 转换函数对于 inline 转换语法非常方便。当然，任何其他 UTF-8/UTF-16 转换代码都可以。
* 只使用接受宽字符（LPWSTR）的 Win32 函数，绝不使用那些接受 LPTSTR 或 LPSTR 的函数。用这种方法传递参数：
 
 `::SetWindowTextW(widen(someStdString or "string litteral").c_str())`

实践 (Policy) 使用下面描述的转换函数。参考，[一个关于转换效率的笔记](http://utf8everywhere.org/#faq.cvt.perf)   

* 对于 MFC 中的字符串：
    ```
    CString someoneElse; // something that arrived from MFC.

    // Converted as soon as possible, before passing any further away from the API call:
    std::string s = str(boost::format("Hello %s\n") % narrow(someoneElse));
    AfxMessageBox(widen(s).c_str(), L"Error", MB_OK);
    ```

* 对于.NET开发者: 使用原始的基于 UTF-16 的字符串类也许很难避免。记住这个实现细节通过类的接口有重大缺陷。
举个例子，string[index] 操作也许会返回一个字符的部分（就像处理一个 UTF-8 字节序列）。当序列化字符串到文件或通讯设备的时候，记得明确 Encoding.UTF8。准备好为了转换付出效率代价。例如，在产生 UTF-8 HTML输出的 ASP.NET web应用中。
    
在 Windows 上处理文件，文件名和字符流

* 总是生成 UTF-8 编码的文本输出文件。
* 因为 [RAII/OOD](http://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization) 的原因，使用 fopen()  应该被避免。但是如果必要，按照上面描述的方式使用 _wfopen() 和 WinAPI。 
* 永远不要传递 std::string 和 const char* 文件名参数给 fstream 家族。MSVC CRT 不支持 UTF-8 参数，但是它有一个非标准的扩展应该按下面的方式使用。
* 使用 widen 将 std::string 参数转化为 std::wstring 

  `std::ifstream ifs(widen("hello"), std::ios_base::binary):`

  当 MSVC 对于 fstream 的态度变化的时候，我们将不得不手动去除该转换。
* 该代码不是多平台的并且在将来也许会不得不手动改变。
* 也可以选择使用一组封装去隐藏转化。

转化函数

这份指南使用的转化函数来自 [Boost.Nowide library](http://cppcms.com/files/nowide/html/)(还不是 boost 的一部分)：
```
std::string narrow(const wchar_t *s);
std::wstring widen(const char *s);
std::string narrow(const std::wstring &s);
std::wstring widen(const std::string &s);
```
这个库也为常用的处理文件和使用 iostreams 读写 UTF-8 的C 和 C++ 库函数提供了一组封装。

这些函数和封装通过 Windows 的 MultiByteToWideChar 和 WideCharToMultiByte 函数很容易实现。任何其它
（也许更快的）转化方法也可以使用。

##11 常见问题

1.问：你是一个 Linux 使用者吗？这是一场隐藏的反对 Windows 的宗教战争吗？

答：不，我使用 Windows 长大，并且我基本上是一个 Windows 开发者。我认为他们在文本方面做出了一个错误的选择，因为他们比其他人做的都早。 —— Pavel

2.问：你是一个亲英派码？你是否私下认为英语字母和文化比其它要优越？

答：不，并且我的国家也并不是讲 ASCII 上字符的语言的国家。我不认为使用一个字节编码 ASCII 字符的编码是盎格鲁中心主义思想，或者与人们沟通有任何的关系。即使一个人可以争辩说程序，网页和 XML 文件，操作系统文件名和其它电脑和电脑的接口本不应该存在，只要他们存在，文本就不仅仅是给人类读者的。

3.问：你们这些人为什么在意？我使用 C# 和/或 Java 编程，我一点也不在意编码。

答：不对。值得庆祝的是，C# 和 Java 都提供 16 位，小于 Unicode 字符集的 char 类型。.NET 索引 str[i] 工作在内部表示单元，因此又出现一个有漏洞的抽象。子串方法将会返回一个无效的字符串，将一个非 BMP 字符分成几部分。

此外，当你将文本写入到磁盘，网络通信，外部设备或其他任何程序会读取地方上的文件的时候，你不得不在意编码。
不要管任何对内容的假定，在这些情况一定要使用 System.Text.Encoding.UTF8(.Net) ，不要使用 Encoding.ASCII，UTF-16 或者手机PDU。

Web 框架像 ASP.NET 也受制于框架之下内部字符串表示选择的匮乏：一个网络应用预期的字符串输出（或者输入）几乎总是 UTF-8 编码，导致在高输入输出的网络应用和网络服务中面临大量的转换。

4.问：UTF-8 不就是一个尝试兼容 ASCII 的一个尝试吗？为什么要保持这个老化石?

答：不管 UTF-8 是不是作为一个兼容的 hack 被发明出来。现在，它是一个比其它任何编码都优秀和流行的编码。

5.问：UTF-16 字符使用超过两个字节的情况在现实世界中非常少见。在实际中这使得UTF-16成为一种固定宽度的编码，使UTF-16有很多优势。我们可以忽略那些字符吗？

答：你真的想在你的软件设计中不支持全部的 Unicode 字符？并且，如果你到底还是想支持它，非 BMP 字符实际上很少改变什么，除了让软件测试更加困难。然而，重要的是相对于仅仅传递字符串，文本操控在现实应用中相对少见。这意味着“几乎固定长度”几乎没有任何性能优势（参考性能），尽管有比较短的字符串也许很有意义。

6.问：为什么不让程序员在内部使用他们最喜欢的编码，只要他们知道如何去使用？

答：我们不反对正确使用任何编码。但是，当相同的类型，例如 std::string 在不同的语境下表示不同的类型的时候，这就成为了一个问题。当为一些人使用 ‘ANSI字符页’，这就意味着该代码是不完整的并且不支持非英文字符文本。在我们的程序中，这意味着 Unicode 意识的 UTF-8 字符串。这种多样性是许多问题和许多灾难的来源：这额外的复杂性并不是这个世界真的需要的一些东西，并且结果就是许多遍布工业界的不支持 Unicode 的糟糕软件。 

7.问：我的应用只是一个 GUI 应用。它没有 IP 通讯或者文件 IO。为什么为了 Windows API 调用要来回转换字符串，而不是单纯地使用宽字符变量？

答：这是一个有效的捷径。事实上，这也许是一个使用宽字符的合适情况。但是如果你计划将来添加一些配置或者一个 log 文件，请考虑将整个东西转成使用窄字符。那将在未来证明正确。

8.问：如果你不想使用 Windows 的 LPTSTR/TCHAR/ETC 宏，为什么要打开 UNICODE 定义？

答：这是为了防止将一个 UTF-8 char* 字符串用到使用预期使用 ANSI 编码的 Windows API 函数中。我们想在这种情况下产生一个编译错误。这是一个和在 Windows 上传递一个 argv[] 字符串到 fopen()上一样难以发现的错误：在这种情况下系统假设用户不会传递一个非当前字符页的文件名。你将很难发现这种错误通过手动测试，除非你的测试经过训练，偶尔使用中文文件名，并且这是一个烂的程序逻辑。得益于使用 UNICODE 定义，你可以因此得到一个错误。

9.问：是不是认为微软某一天将会停止使用宽字符的想法很幼稚？

答：让我们首先看他们什么时候开始支持 CP_UTF-8 做一个一个确定的locale.这应该很难做到。那时候，我们看不到任何人们继续使用宽字符 APIs 的任何理由。并且，增加对 CP_UTF8 的支持也不会改变现存的不支持 UNICODE 的程序和库。

一些人认为增加 CP_UTF8 的支持将会破坏现在使用 ANSI API的应用。这也被认为是微软不得不创造宽字符 API的原因。这不正确，即使一些流行的 ANSI 编码也是变长的（举例来说，Shift JIS）,所以并没有正确的代码会被破坏。微软选择 UCS-2 的原因完全是历史决定的。在那个时候，UTF-8 还并没有存在，Unicode 仅仅被认为是“一个宽的 ASCII”，使用固定长度的编码被认为是很重要的。

8.问：字符 (characters)，码点(code points)，码元(code units)和象形聚簇(grapheme clusters)是什么？

10.问：你如何看待字节顺序标记（Byte Order Marks）？

答：根据 Unicode 标准（v6.2,p.30）:对于 UTF-8，既不要求也不建议使用 BOM 。

字节顺序也是避免使用 UTF-16 的另一个原因。UTF-8 没有端性问题，并且 UTF-8 BOM 存在只是为了表明这是一个 UTF-8 流。 如果将来 UTF-8 是唯一流行的编码（在互联网世界中已经是这样），BOM 就会变得冗余。实践中，大多数 UTF-8 文本文件已经省略了 BOMs。

使用 BOMs 需要所有现存的代码知道它们，即使像在文件串接等很简单的情景中。这是不可接受的。

11.问：你如何看待行尾字符 (line endings) ？

答：总是使用 \n(0x0a) 行尾字符，即使是在 Windows 上。文件应该在二进制模式读和写，因为这保证了互通性 —— 在任何系统上，一个程序将总是会给出相同的输出。因为 C 和 C++ 标准使用 \n 作为内存中的行尾，这将导致所有文件用 POSIX 方式写入。当文件在 Windows 上用记事本打开的时候，这将会导致问题；但是，任何像样的文本阅读器都理解这样的行尾。

我们同样很欣赏 SI 单元。ISO-8601日期格式，和浮点类型的小数点(and floating point to the floating comma)。

12.问：那文本处理算法的性能，字节对齐这些问题呢？

答：使用 UTF-16 真的更好吗？也许是这样。ICU 因为历史的原因使用 UTF-16,因此很难去衡量。然而，大多数字符串被看做 cookies, 当再次用的时候既不排序也不反转。更小的字符编码也许更有利于性能。

13.问：人们经常误用 UTf-16，认为它每个字符占据 16 个字节,这真的是它的过错吗？

答：不是，但是安全性是每一种设计的重要特性。

14.问：如果 std::string 表示 UTF-8，那不会和在 std::strings 中存储纯文本的代码弄混吗？

答：根本没有叫做纯文本的东西，根本没有任何理由在一个叫 string 的类中只保存 ANSI 代码页或者仅含 ASCII 字符的文本。

15.问：当向窗口传递字符串的时候，UTF-8 和 UTF-16 编码之间的转换不会使我的程序变慢吗？

答：首先，无论怎样你都要做一些转换。无论是调用系统调用，还是和外界交流。即使在你的程序中你和系统的交互更频繁，这里有一个小小的实验。

系统的一个典型应用是打开文件，这个函数在我的机器上执行了（184±3）μs:
```
void f(const wchar_t* name)
{
    HANDLE f = CreateFile(name, GENERIC_WRITE, FILE_SHARE_READ, 0, CREATE_ALWAYS, 0, 0);
    DWORD written;
   WriteFile(f, "Hello world!\n", 13, &written, 0);
   CloseHandle(f);
}
```
而这个执行了（186±0.7）μs：
```
void f(const char* name)
{
    HANDLE f = CreateFile(widen(name).c_str(), GENERIC_WRITE, FILE_SHARE_READ, 0, CREATE_ALWAYS, 0, 0);
    DWORD written;
    WriteFile(f, "Hello world!\n", 13, &written, 0);
    CloseHandle(f);
}
```
(在这两个情况中，都使用 name="D:\\a\\test\\subdir\\subsubdir\\this is the sub dir\\a.txt" 运行。平均运行超过了5次。我们使用了一个优化过的依赖于 C++11中 std::string 临近存储的 widen 。)

这仅仅多了（1±2）%。此外，MultiByteToWideChar 肯定不是最快的 UTF-8 ↔ UTF-16 转换函数。

16. 问：如何在我的 C++ 代码中写入 UTF-8 字符串？

答：如果你想国际化你的软件，那么所有的非 ASCII 字符串都会从一个外部翻译数据库加载，所以这不是一个问题。 如果你依然想嵌入特殊的字符，你可以按照下面的做。在 C++11 中，你可以这样做：

                        u8"∃y ∀x ¬(x ≺ y)"

对于不支持 'u8' 的编译器，你可以按下面硬编码 UTF-8 的码元：

                        "\xE2\x88\x83y \xE2\x88\x80x \xC2\xAC(x \xE2\x89\xBA y)"

但是最直接的方法是就按照字符串来写并且用 UTF-8 编码保存源文件。

                        "∃y ∀x ¬(x ≺ y)"

不幸的是，MSVC 将它转化成一些 ANSI 内码表，使字符串出错。为了解决这个问题，用没有 BOM 的 UTF-8 保存文件。MSVC 就会认为这是正确的内码表，就不会改变你的字符串。然而，这使它无法使用 Unicode 标识符和宽字符串（总之你不会使用它）。

17.问：我有一个复杂的大的基于 char 的 Windows 应用，让它可以感知 Unicode 的最简单方法是什么？

答: 保留着 chars。定义 UNICODE 和 _UNICODE ，使得在应该使用 narrow()/widen() 的地方得到编译错误（通过在 Visual Studio 项目设置中选择使用 Unicode 字符集，这会自动完成）。找到所有使用 fstream 和 fopen() 的地方，按照上面描述的重载宽字符。现在，你几乎完成了。

如果你使用不支持 Unicode 的第三方库，例如直接转发文件名字符串到 fopen()，你将不得不使用上面显示的 GetShortPathName() 工具解决这个问题。

18.问：Python 呢？我听说他们在 3.3 版本中为了更好的支持 Unicode 非常努力。

答：也许他们应该做少一点，这样支持会更好。在 3.3 版本的 CPython 参考实现中，内部字符串表示做了改动。根据字符串内容的不同，UTF-16 编码被三种可能的编码 （ISO-8859-1,UCS-2或者 UCS-4）之一取代。增加一个非 ASCII 或者 非 BMP 字符，整个字符串将会经常隐形地转换为不同的编码。内部编码对于脚本来说是透明的。这个设计是为了在Unicode码点上优化索引操作的性能。但是，我们认为相对于字形族，计数或者索引码点对于大多数的使用并不重要。据我们所知，Python 现今并不对后者提供支持。

因此，我们反对字符串不可知的表示处理方式，赞成使用UTF-8内部编码表示的透明表示的API。索引操作应该计算码元而不是码点，正如他们改变前做的那样。这样也将简化实现，并且提高性能，例如，在已经被 UTF-8 编码文本统治的 Web 的脚本上，这样可以使 Python 编程语言在服务端领域更加可用。一些人也许会认为脚本程序员字符剪切操作的安全性，但又说回来，对于分开象形族，这样的认为是有效的。虽然现在已经全面支持 Unicode，我们相信 Python，作为一个有很少历史负担的现代工具，一定能在文本处理领域做的更好。

除了这些，JPython 和 IronPython 依然依赖于它们寄主平台（分别是 Java 和 .NET）的不够好的编码，并且为了正确的处理代理对，一定要小心。

19.问；但是为什么 std::string？它不会是一个更好的，基于对象的感知 UTF-8 的字符串类？

答：并不是处理字符串的每一块代码都实际上涉及到文本的处理和确认。一个需要接收 Unicode 文件名并且传递给文件 IO 的文件复制程序，能够很好的处理简单字节缓冲区。如果你设计一个接收字符串的库，简单标准和轻量的 std::string 就足够。相反，重新发明一个新的字符串类并且强迫每个人使用你的奇特的接口是错误的。当然，如果一个人需要的不仅仅是传递字符串，他应该使用合适的文本处理工具。

20.问：我已经使用了这个方法并且我想让我们的目标实现，我能做些什么？

答：传播下去。

检查你的代码，了解在一个可移植的，支持 Unicode 的代码中，哪些库用起来最痛苦。给作者发一份 bug 报告。如果你是一个 C 或者 C++ 库作者，使用支持 UTF-8 的 char* 和 std::string，并且拒绝支持 ANSI 内码表 —— 因为他们本质上是不支持 Unicode 的。

如果你是一个微软的员工，推动支持CP_UTF8 作为一个窄 API 代码页。

进一步的建议:

* 建立一个程序库，存放使常用的第三方库（如PugiXML，LibTIFF等）支持 UTF-8 的补丁
。给 Windows 上的标准库函数（例如 fopen()）建立一个链接时补丁，完成恰当的参数转换。这个可以为 main() 函数和全局变量实现。

##12 关于作者

这rkus Künne, Jelle Geerts and Lazy Rui for reporting bugs and typos in this document.份宣言的作者是 [Pavel Radzivilovsky](http://stackoverflow.com/users/73656/pavel-radzivilovsky),[Yakov Galka](http://stannum.co.il/about) 和 [Slava Novgorodov](http://slavanov.com/)。这是我们的经验和对现实世界中 Unicode 的问题和对程序员在现实中犯的错误的调查的结果。目标是提高对文本问题的意识并启发工业范围内的变革，使得支持 Unicode 编程更加容易，最终提升使用人类工程师写的那些程序的用户的体验。我们中的任何人都没有参与到 Unicode 委员会。

特别感谢 Glenn Linderman 提供关于 Python 的信息, Markus Künne, Jelle Geerts and Lazy Rui 报告文档中的错误和排版错误。

许多文本来自于 Boost.Locale 的作者 [Artyom Beilis 在 StackOverflow 上发起的讨论](http://programmers.stackexchange.com/questions/102205/should-utf-16-be-considered-harmful)。另外的启发来自 [VisionMap](http://www.visionmap.com/) 的发展会议和 Micheal Hartl 的 [tauday.org](http://tauday.com/tau-manifesto)。

##13 外部链接

* [Unicode 委员会](http://www.unicode.org/) (Unicode 标准，[PDF](http://www.unicode.org/versions/Unicode6.2.0/UnicodeStandard-6.2.pdf))
* [International Components for Unicode ](http://site.icu-project.org/)(ICU)
* [Joel on Unicode](http://www.joelonsoftware.com/articles/Unicode.html)——‘The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets’
* [Boost.Locale](http://cppcms.sourceforge.net/boost_locale/html/)—— C++实现的高质量本地化工具
* [Should UTF-16 be considered harmful     ](http://programmers.stackexchange.com/questions/102205/should-utf-16-be-considered-harmful)Artyom Beilis 在 StackOverflow 上发起的讨论。
* [How twitter counts characters](https://dev.twitter.com/docs/counting-characters)

##14 反馈

你可以在 Facebook [UTF-8 Everywhere](https://www.facebook.com/UTF8Everywhere) 页面评论或反馈。非常感谢你的帮助和反馈。

![](https://github.com/Mjinrui/UTF-8Everywhere/blob/master/utf8donate.png)

比特币捐赠到：1UTF8gQmvChQ4MwUHT6XmydjUt9TsuDRn

该款项将会用于研究和推广。

