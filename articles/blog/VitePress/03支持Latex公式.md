参考：https://blog.csdn.net/delete_you/article/details/130815350

目的：支持Latex数学公式

分析：由于目前vitepress最新版依旧采用markdown-it作为md渲染库，默认并不支持latex语法，所以我们需要将其替换为另一个渲染库才可以。

这里使用最新的 `markdown-it-mathjax3` 作为渲染库。

先安装。

```
npm install markdown-it-mathjax3 -D
或者
yarn add markdown-it-mathjax3 -D
```

打开文件夹 `.vitepress/config.js` 添加如下代码即可

```
import mathjax3 from "markdown-it-mathjax3";

const customElements = [
	"math",
	"maction",
	"maligngroup",
	"malignmark",
	"menclose",
	"merror",
	"mfenced",
	"mfrac",
	"mi",
	"mlongdiv",
	"mmultiscripts",
	"mn",
	"mo",
	"mover",
	"mpadded",
	"mphantom",
	"mroot",
	"mrow",
	"ms",
	"mscarries",
	"mscarry",
	"mscarries",
	"msgroup",
	"mstack",
	"mlongdiv",
	"msline",
	"mstack",
	"mspace",
	"msqrt",
	"msrow",
	"mstack",
	"mstack",
	"mstyle",
	"msub",
	"msup",
	"msubsup",
	"mtable",
	"mtd",
	"mtext",
	"mtr",
	"munder",
	"munderover",
	"semantics",
	"math",
	"mi",
	"mn",
	"mo",
	"ms",
	"mspace",
	"mtext",
	"menclose",
	"merror",
	"mfenced",
	"mfrac",
	"mpadded",
	"mphantom",
	"mroot",
	"mrow",
	"msqrt",
	"mstyle",
	"mmultiscripts",
	"mover",
	"mprescripts",
	"msub",
	"msubsup",
	"msup",
	"munder",
	"munderover",
	"none",
	"maligngroup",
	"malignmark",
	"mtable",
	"mtd",
	"mtr",
	"mlongdiv",
	"mscarries",
	"mscarry",
	"msgroup",
	"msline",
	"msrow",
	"mstack",
	"maction",
	"semantics",
	"annotation",
	"annotation-xml",
	"mjx-container",
	"mjx-assistive-mml",
];

export default {
	markdown: {
		config: (md) => {
			md.use(mathjax3);
		},
	},
	vue: {
		template: {
			compilerOptions: {
				isCustomElement: (tag) => customElements.includes(tag),
			},
		},
	},
};
```

latex测试

> F = \sum_{n=-\infty}^{\infty}\left|\mathscr{F}\left[f(x)\right]\right|^{2}\Delta x

$$F = \sum_{n=-\infty}^{\infty}\left|\mathscr{F}\left[f(x)\right]\right|^{2}\Delta x$$

