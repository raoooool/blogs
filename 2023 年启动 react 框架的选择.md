#React #脚手架

2023 年，React 团队在新的 React 框架文档中正式启用了 Create React App 脚手架，转而推荐了几个工具集。作者对比了以下几个框架：

#### 一、Vite

Vite 底层使用了 esbuild 和 esm，所以开发和打包特别快。

优点：轻量、 SSR 可选、和 React 不强绑定、更好地理解 React 底层配置；  
缺点：对 SPA/CSR 优先、对 React Server Component 支持不好。  

#### 二、Next.js  

next 的大名不用多说，已经被 React 官方列为默认框架了。最大的亮点是支持混合渲染，比如一部分页面使用 SSG、进入到应用程序后使用 SSR。  

优点：SSR 默认、和 React 官方团队关系很近、拥有最前沿的技术；  
缺点：拥有最前沿的技术、和 Vite 相比，不够快、学习曲线比 react 陡峭，并且距离 react 底层封装太多，不利于学习。  

#### 三、Astro  

Gatsby 的竞争对手，专注于轻交互的内容网站。

优点：性能好；SEO 好；不和 eract 强绑定。  
缺点：对动态网站支持不好。

#### 总而言之

要简单，选 Vite。要全面，选 Next。如果只构建静态网站，选 Astro。

---

<https://www.robinwieruch.de/react-starter/>
