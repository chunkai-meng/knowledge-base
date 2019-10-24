# Mathematical Formula in R Markdown


> This is a cheat sheet I reorganized, it's aim all the Symbols or Greek letters could be appear in the R markdown, some of them can also be use in reglar markdown files. [[Click]](http://rpubs.com/Chunkai/Mathematical-Formula-in-R-Markdown)


> 部分内容整理自<a href="http://oiltang.com/2014/05/04/markdown-and-mathjax/">Oil Tang</a>  
> 参考[MathJax 快速参考](http://colobu.com/2014/08/17/MathJax-quick-reference/)
> 希腊字母表来自维基百科  
> Maintainer: Chunkai

# 如何插入公式

LaTeX的数学公式有两种：行中公式和独立公式。行中公式放在文中与其它文字混编，独立公式单独成行。

行中公式可以用如下两种方法表示：

＼(数学公式＼)　或　\$数学公式\$（要把人民币符号换成美元符号）

独立公式可以用如下两种方法表示：

＼[数学公式＼]　或　\$ \$数学公式\$ \$（要把人民币符号换成美元符号）

例子：  
J\\alpha(x) = \\sum{m=0}^\\infty \\frac{(-1)^m}{m! \\Gamma (m + \\alpha + 1)} {\\left({ \\frac{x}{2} }\\right)}^{2m + \\alpha}

显示：
$$
J\alpha(x) = \sum{m=0}^\infty \frac{(-1)^m}
{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha}
$$

# 如何输入上下标

\^表示上标, \_表示下标。如果上下标的内容多于一个字符，要用\{\}把这些内容括起来当成一个整体。上下标是可以嵌套的，也可以同时使用。

例子：x^{y^z}=(1+{\\rm e}^x)^{-2xy^w}

显示： $$x^{y^z}=(1+\rm {e}^x)^{-2xy^w}$$

另外，如果要在左右两边都有上下标，可以用\sideset命令。

例子：\sideset{^12}{^34}\bigotimes

显示： $$\sideset{^12}{^34}\bigotimes$$

# 如何输入括号和分隔符

()、[]和|表示自己，{}表示{}。当要显示大号的括号或分隔符时，要用\\left和\\right命令。

例子：f(x,y,z) = 3y^2z \\left( 3+\\frac{7x+5}{1+y^2} \\right)

显示： $$f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right)$$

有时候要用 \\left. 或 \\right. 进行匹配而不显示本身。

例子：\\left. \\frac {\\rm du} {\\rm dx} \\right| \_{x=0}
\\rm 空格 可以改变其后的text格式为正常格式（默认公式中的字母为斜体）,其他格式可以参照#13
显示：  $$\left. \frac{ {\rm d}u}{ {\rm d}x} \right| _{x=0}$$

# 如何输入分数

例子：\\frac{1}{3}　或　1 \\over 3

显示： $\frac{1}{3}$ 　或   $1 \over 3$

# 如何输入开方

例子：\\sqrt{2}　和　\\sqrt[n]{3}

显示： $\sqrt{2}$ 　和　 $\sqrt[n]{3}$

# 如何输入省略号

数学公式中常见的省略号有4种，
\\ldots 表示与文本底线对齐的省略号，  
\\cdots 表示与文本中线对齐的省略号,   
\\vdots 表示纵向排列的省略号,   
\\ddots 表示西方向排列的省略号。

例子：f(x1,x2,\\ldots,xn) = x1^2 + x2^2 + \\cdots + xn^2

显示： $$f(x1,x2,\ldots,xn) = x1^2 + x2^2 + \cdots + xn^2$$

# 如何输入矢量
例子：\\vec{a} \\cdot \\vec{b}=0

显示： $$\vec{a} \cdot \vec{b}=0$$

# 如何输入积分

例子：\\int_0^1 x^2 {\\rm d}x

显示： $$\int_0^1 x^2 {\rm d}x$$

# 如何输入极限运算

例子：\\lim_{n \\rightarrow +\\infty} \\frac{1}{n(n+1)}

显示： $$\lim_{n \rightarrow +\infty} \frac{1}{n(n+1)}$$

# 如何输入累加、累乘运算

例子：\\sum \_{i=0}^n \\frac{1}{i^2 }　和　\\prod \_{i=0}^n \\frac{1}{i^2}

显示：
$$
\prod_{i=0}^n \frac{1}{i^2}
$$

和
$$\sum_{i=0}^n \frac{1}{i^2}$$

# 如何进行公式应用
例子：＼begin{equation}\\label{equation1}r = rF+ \\beta(rM – r_F) + \\epsilon＼end{equation}

显示：$$\begin{equation}\label{equation1}r = rF+ \beta(rM – r_F) + \epsilon\end{equation}$$

引用：请见公式( \ref{equation1} )

12．希腊字母：  

字母 | 表示 | 大写 | 表示 | 发音(英/美)
---|---|---|---|---
$\alpha$ | \\alpha | $A$ | A | /ˈælfə/
$\beta$ | \\beta | $B$ | B | /ˈbiːtə/	/ˈbeɪtə/
$\gamma$ | \\gamma | $\Gamma$ | \\Gamma | /ˈɡæmə/
$\delta$ | \\delta | $\Delta$ | \\Delta | /ˈdɛltə/
$\epsilon$ | \\epsilon | $E$ | E | /ɛpˈsaɪlən/	/ˈɛpsɨlɒn/
$\varepsilon$ | \\varepsilon |  |  |
$\zeta$ | \\zeta | $Z$ | Z | /ˈziːtə/	/ˈzeɪtə/
$\eta$ | \\eta | $E$ | E | /ˈiːtə/	/ˈeɪtə/
$\quad$ |   |   |    |   
$\theta$ | \\theta | $\Theta$ | \\Theta |/ˈθiːtə/	/ˈθeɪtə/
$\vartheta$ | \\vartheta |  |  |
$\iota$ | \\iota | $I$ | I | /aɪˈoʊtə/
$\kappa$ | \\kappa | $K$ | K |/ˈkæpə/
$\lambda$ | \\lambda | $\Lambda$ | \\Lambda |/ˈlæmdə/
$\mu$ | \\mu | $M$ | M |/ˈmjuː/	/ˈmuː/
$\nu$ | \\nu | $N$ | N |/ˈnjuː/	/ˈnuː/
$\xi$ | \\xi | $\Xi$ | \\Xi |/ˈzaɪ/, /ˈksaɪ/
$\quad$ |   |   |  
$o$ | \\o | $O$ | O | /oʊˈmaɪkrɒn/
$\pi$ | \\pi | $\Pi$ | \\Pi |/ˈpaɪ/
$\varpi$ | \\varpi |  |
$\rho$ | \\rho | $P$ | P |/ˈroʊ/
$\varrho$ | \\varrho |  |  |
$\sigma$ | \\sigma | $\Sigma$ | \\Sigma |/ˈsɪɡmə/
$\varsigma$ | \\varsigma |  |
$\tau$ | \\tau | $T$ | T |/ˈtaʊ/，/ˈtɔː/
$\quad$ |   |   |  |
$\upsilon$ | \\upsilon | $\Upsilon$ | \\Upsilon |/juːpˈsaɪlən/ /ʌpˈsaɪlən/
$\phi$ | \\phi | $\Phi$ | \\Phi |/ˈfaɪ/
$\varphi$ | \\varphi |  |  |
$\chi$ | \\chi | $A$ | A |/ˈkaɪ/
$\psi$ | \\psi | $\Psi$ | \\Psi | /ˈsaɪ/，/ˈpsaɪ/
$\omega$ | \\omega | $\Omega$ | \\Omega |/ˈoʊmɨɡə/ /oʊˈmeɪɡə/


# 常用符号

## 关系运算符：  
$\pm$ ：\\pm

$\times$ ：\\times

$\div$ ：\\div

$\mid$ ：\\mid

$\nmid$ ：\\nmid

$\cdot$ ：\\cdot

$\circ$ ：\\circ

$\ast$ ：\\ast

$\bigodot$ ：\\bigodot

$\bigotimes$ ：\\bigotimes

$\bigoplus$ ：\\bigoplus

$\leq$ ：\\leq

$\geq$ ：\\geq

$\neq$ ：\\neq

$\approx$ ：\\approx

$\sim$  : \\sim

$\cong$ : \\cong

$\equiv$ ：\\equiv

$\sum$ ：\\sum

$\prod$ ：\\prod

$\coprod$ ：\\coprod

##集合运算符：  
$\emptyset$ ：\\emptyset

$\in$ ：\\in

$\notin$ ：\\notin

$\subset$ ：\\subset

$\supset$ ：\\supset

$\subseteq$ ：\\subseteq

$\supseteq$ ：\\supseteq

$\bigcap$ ：\\bigcap

$\bigcup$ ：\\bigcup

$\bigvee$ ：\\bigvee

$\bigwedge$ ：\\bigwedge

$\biguplus$ ：\\biguplus

$\bigsqcup$ ：\\bigsqcup

##对数运算符：  
$\log$ ：\\log

$\lg$ ：\\lg

$\ln$ ：\\ln

##三角运算符：  
$\bot$ ：\\bot

$\angle$ ：\\angle

$30^\circ$ ：30^\\circ

$\sin$ ：\\sin

$\cos$ ：\\cos

$\tan$ ：\\tan

$\cot$ ：\\cot

$\sec$ ：\\sec

$\csc$ ：\\csc

##微积分运算符：  
$\prime$ ：\\prime

$\int$ ：\\int

$\iint$ ：\\iint

$\iiint$ ：\\iiint

$\iiiint$ ：\\iiiint

$\oint$ ：\\oint

$\lim$ ：\\lim

$\infty$ ：\\infty

$\nabla$ ：\\nabla

$\bigtriangleup$  : \\bigtriangleup

$\bigtriangledown$  : \\bigtriangledown

$\S$  : \\S

##逻辑运算符：  
$\because$ ：\\because

$\therefore$ ：\\therefore

$\forall$ ：\\forall

$\exists$ ：\\exists

$\not=$ ：\\not=

$\not>$ ：\\not>

$\not\subset$ ：\\not\subset

##戴帽符号：  
$\hat{y}$ ：\\hat{y}

$\check{y}$ ：\\check{y}

$\breve{y}$ ：\\breve{y}

##连线符号：  
$\overline{a+b+c+d}$ ：\\overline{a+b+c+d}

$\underline{a+b+c+d}$ ：\\underline{a+b+c+d}

$\overbrace{a+\underbrace{b+c}{1.0}+d}^{2.0}$ ：\\overbrace{a+\underbrace{b+c}{1.0}+d}^{2.0}

##箭头符号：  
$\uparrow$ ：\\uparrow

$\downarrow$ ：\\downarrow

$\Uparrow$ ：\\Uparrow

$\Downarrow$ ：\\Downarrow

$\rightarrow$ ：\\rightarrow

$\leftarrow$ ：\\leftarrow

$\Rightarrow$ ：\\Rightarrow

$\Leftarrow$ ：\\Leftarrow

$\longrightarrow$ ：\\longrightarrow

$\longleftarrow$ ：\\longleftarrow

$\Longrightarrow$ ：\\Longrightarrow

$\Longleftarrow$ ：\\Longleftarrow

## 函数名
latex公式中字母默认为变量，用斜体排版，但是函数名通常用罗马字体正体排版，因此，可以用一下函数名符号书写  
\\sin + \\cos + \\tan + \\max + \\min + \\lim + \\log \\ln + \\deg + \\exp
$$\sin + \cos + \tan + \max + \min + \lim + \log \ln + \deg + \exp$$

## 要输出字符
$$\\# \quad \$　\%　\&　\\_　\\{　\\}$$

# 公式编排
## 列表
用\- 作为无序列表的标记，不要用*
## 空格  
\\quad 生成一个空格（相当于大写’M’的宽度）  
\\qquad 生成一个大空格  
\\, 相当于3/18个\\quad  
: 相当于4/18个\\quad  
\\; 相当于5/18个\\quad  
! 生成一个负空格-3/18个\\quad  

## 分隔符（括号）
(), [], {}, <>等均可以作为分隔符(分隔符主要指括号)  
但是{}要用\{,\}表示

## 分割符大小调整
- 将\\left放在分隔符前，tex会自动调整分隔符的大小  
- 但是每个\\left必须要用一个\\right关闭  
- 如果分隔符仅有左括号则用\\right关闭  
如：\\left(\\frac{\\sqrt x}{y^3}\\right)
显示效果为:
$$\left(\frac{\sqrt x}{y^3}\right)$$

## 对齐数组/矩阵
使用\\begin{array}{}和\\begin{matrix}都可以整齐排列公式等，个人喜欢用array,如下：

$$ \left[
\begin{array}{cc|c}
  1 & 2 & 3 \\\\
  4 & 5 & 6
\end{array}
\right] $$

$$
    \begin{matrix}
    1 & x & x^2 \\\\
    1 & y & y^2 \\\\
    1 & z & z^2 \\\\
    \end{matrix}
$$
多级列表的缩进要连续两个tab再加标记-/*

- 列由&分隔
- 行由\\分隔
- 列样式有n(列数）个表示列对齐方式的字母组成，字母意义如下：
    - l 该列左对齐排列
    - c 该列居中排列
    - r 该列右对齐排列

$$
X=
\left( \begin{array}{ccc}
x_1 & x_2 & \cdots \\\\
x_3 & x_4 & \cdots \\\\
\vdots & \vdots & \ddots
\end{array}\right)
$$

# 如何进行字体转换  
\\mathrm{$\mathrm{正常字体}$}  
\\mathit{$\mathit{斜体}$}  
\\mathbf{$\mathbf{粗体符号boldfont}$}  
\\mathbb{$\mathbb{空心粗体blackboard}$}  
\\mathcal{$\mathcal{花体 flower}$}  
\\mathsf{$\mathsf{等线体}$}
\\mathtt{$\mathtt{打字机体}$}
