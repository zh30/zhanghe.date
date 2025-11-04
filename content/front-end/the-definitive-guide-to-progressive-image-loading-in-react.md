---
title: React 渐进式图像加载权威指南：架构、实现与性能权衡
date: 2025-11-04
---

## 第一部分：解构“渐进式加载”：一个战略性分类

在现代前端开发中，“渐进式加载”是一个被频繁提及但又容易引起混淆的术语。它并非指代单一技术，而是涵盖了一系列旨在优化网页图像加载性能和用户体验的策略。这些策略的核心目标是改善真实的和感知的加载速度 1。为了在 React 应用中做出明智的架构决策，我们必须首先对这个模糊的术语进行精确的分类。

本报告将“渐进式加载”分解为三个截然不同但又相互关联的技术领域，它们分别回答了图像加载过程中的“何时”、“期间”和“如何”的问题。

### 分类一：延迟加载 (Lazy Loading) - “何时”加载

延迟加载是一种策略，它将非关键资源（如下文的图像）的加载推迟到需要它们时才执行 2。

- **传统实现：** 历史上，这需要通过监听 `scroll` 和 `resize` 事件来实现，这种方法既不精确又容易导致主线程阻塞，引发页面卡顿 4。

- **现代实现：** 现代浏览器提供了两种高效的解决方案。首先是 `IntersectionObserver` API，它提供了一种异步、高性能的方式来检测元素何时即将进入视口，而不会阻塞主线程 5。其次是原生的 HTML `loading="lazy"` 属性，它将延迟加载的逻辑完全委托给浏览器 2。


### 分类二：低质量图像占位符 (LQIP) - “期间”显示什么

LQIP 是一种专注于改善 _感知性能_ 的技术。它通过在全分辨率图像下载完成前，立即显示一个低保真度的视觉预览来填充图像空间 1。

- **占位符类型：**

    1. **纯色占位符：** 最简单的方式，通常提取图像的主色调作为背景色 8。

    2. **“模糊上升”(Blur-Up) 效果：** 由 Medium 推广的流行技术 6。它使用一个极小的（例如 < 2kB）、通常内联为 Base64 字符串的图像版本，然后通过 CSS `filter: blur()` 属性将其放大并模糊化 1。

    3. **BlurHash：** 一种更先进的技术，它将图像的模糊预览编码为一个极短的算法字符串（约 20-30 个字符），然后在客户端解码为模糊图像 11。


### 分类三：渐进式渲染 (Progressive Rendering) - “如何”渲染

这与 React 或 JavaScript 无关，而是指图像文件本身的编码和解码方式。

- **基线 JPEG (Baseline JPEG)：** 图像数据按从上到下的顺序存储和渲染。在慢速网络下，用户会看到图像自上而下逐行显示 13。

- **渐进式 JPEG (Progressive JPEG)：** 图像数据被编码为多次扫描。浏览器首先下载并显示一个完整的、低分辨率的图像，然后在后续扫描中逐渐提高其清晰度，直到达到全分辨率 13。这是一个在资产级别进行的优化 16。


### 黄金标准与核心 Web 指标 (Core Web Vitals) 的影响

在实践中，上述三种技术必须结合使用，才能形成一个健壮的解决方案。

- **仅使用延迟加载 (分类一)：** 当图像滚动到视口时，会出现一个空白区域，然后图像突然“弹入”(pop-in)，这种体验是突兀的，并且如果未正确设置尺寸，会直接导致布局位移 (CLS)。

- **仅使用 LQIP (分类二)：** 这会 _恶化_ 初始加载性能，因为浏览器现在需要急切地下载 _两个_ 资源：占位符和全分辨率图像 18。

- **仅使用渐进式 JPEG (分类三)：** 图像仍然是急切加载的，会与页面上的其他关键资源（如 CSS 或 JS）竞争网络带宽。


**理想的策略是将“延迟加载”(分类一) 与“LQIP”(分类二) 相结合。** 这种组合策略首先通过 LQIP 占位符在视觉上保留空间，然后在用户即将看到图像时才触发高分辨率图像的加载。

这种组合方法对 Google 的核心 Web 指标有直接的正面影响：

1. **CLS (Cumulative Layout Shift)：** 这是占位符解决的核心问题。通过为图像（或其占位符容器）提供明确的 `width` 和 `height` 属性（或使用 `aspect-ratio`），可以在高分辨率图像加载完成 _之前_ 就“预留”出正确的空间，从而防止页面布局在图像载入时发生跳动 4。

2. **LCP (Largest Contentful Paint)：** 这是一个需要精细处理的指标。位于“首屏”(above-the-fold) 的图像通常是页面的 LCP 元素。对 LCP 元素使用延迟加载是一个明确的 _反模式_ 18。现代框架（如 Next.js 和 Gatsby）提供了 `priority` 或类似属性，以确保 LCP 图像被立即加载 18。然而，对“首屏以下”(below-the-fold) 的图像应用延迟加载，可以通过减少初始网络竞争来 _改善_ LCP 22。

3. **INP (Interaction to Next Paint) / FID：** 通过推迟非关键图像的加载和解码工作，并避免使用高成本的 `scroll` 事件监听器，主线程可以被释放出来，从而更快地响应用户输入，改善页面的交互性 2。


## 第二部分：框架原生解决方案：抽象化（且推荐）的途径

前端行业已经趋向于一个共识：生产环境下的图像优化是一个极其复杂的问题。它涉及响应式尺寸生成、现代格式（WebP/AVIF）转换、CLS 预防、延迟加载实现和占位符生成 18。手动为每张图片执行此操作是不可持续的。

因此，**图像优化已成为构建工具和框架的核心职责，而不是应用开发者的**。对于新项目，强烈建议使用框架提供的原生解决方案。

### A. Next.js 的 `next/image` 组件：按需优化

`next/image` 组件充当一个按需 (on-demand) 图像优化服务器 19。当用户请求图像时，Next.js 服务器（或 Vercel Edge）会拦截该请求，动态地转换图像（调整大小、转换为 WebP/AVIF），然后提供服务并缓存结果以备将来使用。

**核心特性：**

- 自动调整大小和格式优化 (WebP/AVIF) 19。

- 强制使用 `width`/`height` 或 `fill` 属性以防止 CLS 19。

- 默认启用原生延迟加载 (`loading="lazy"`) 24。

- 通过 `placeholder` 属性支持占位符 (`"empty"` 或 `"blur"`) 24。


#### 深入探讨：`blurDataURL` 的困境

`placeholder="blur"` 的易用性完全取决于图像的来源。这是一个关键的实现细节，常常导致开发者困惑。

1. **本地（静态）图像：** 这是“神奇”的场景。当开发者通过 `import profileImage from './profile.png'` 导入图像时，Next.js 可以在 _构建时_ 访问和处理该文件。它会自动生成一个微小的、模糊的图像，将其编码为 Base64 字符串，并将其作为 `blurDataURL` 注入到组件中 20。

2. **远程（动态）图像：** 这是“手动”的场景。如果 `src` 是来自 CMS 的 URL 字符串（例如 `src="https://.../image.jpg"`），Next.js 在构建时 _无法访问_ 该文件 28。因此，如果设置了 `placeholder="blur"`，Next.js 会 _要求_ 开发者手动提供 `blurDataURL` 属性 25。如果不提供，将不会显示模糊占位符。


#### 实施指南：为远程图像生成 `blurDataURL`

这是将 `next/image` 与 CMS 结合使用时的核心挑战。以下是几种在服务器端（`getStaticProps` 或 `getServerSideProps` 中）生成 `blurDataURL` 的策略：

- **策略一（推荐）：使用 `plaiceholder`**

    - **流程：** 这是一个功能强大的库。在服务器端，获取远程图像的 Buffer，使用 `plaiceholder` 处理它以生成 `base64` 字符串 29，然后将此字符串作为 `blurDataURL` prop 传递给页面组件 30。

    - **注意：** 此过程必须在服务器上运行；`plaiceholder` 无法在客户端运行 32。

- **策略二（性能技巧）：嵌套 Next.js 优化器**

    - **流程：** 这是一种更巧妙的技术。与其在服务器端下载完整的 5MB 原始图像，不如 `fetch` Next.js _自己的_ 图像优化器端点：`fetch('/_next/image?url=...&w=16&q=75')` 33。这将获取一个 16px 宽的超小版本。然后，将这个微小的响应 Buffer 转换为 Base64。

    - **优势：** 比下载完整图像更快，占用服务器带宽更少。

- **策略三（手动）：使用 `sharp`**

    - **流程：** 类似于 `plaiceholder`，但直接使用 `sharp` 库将图像大小调整为约 10px 宽，转换为 Buffer，然后再编码为 Base64 字符串 35。

- **策略四（静态）：手动生成器**

    - **流程：** 对于不常更改的图像（如作者头像），可以使用 `blurred.dev` 这样的在线工具生成一次 Base64 字符串，并将其硬编码在代码中 36。


### B. Gatsby 的 `gatsby-plugin-image`：构建时优化

Gatsby 采取了“预先优化”(ahead-of-time) 的方法。它使用 `gatsby-plugin-sharp` 和 `gatsby-transformer-sharp`，在 _构建时_ 处理 _所有_ 图像，生成所有响应式尺寸和占位符 23。

**组件：`StaticImage` vs. `GatsbyImage`**

- `StaticImage`：用于每次使用时都相同的图像（如 Logo）。`src` 是一个静态的文件路径或 URL 39。

- `GatsbyImage`：用于作为 prop 传入的动态图像（如来自 CMS 的博客文章特色图）39。


#### GraphQL 生态系统

这是 Gatsby 与 Next.js 的根本区别。在 Gatsby 中，开发者通常不只是传递一个 `src` 字符串，而是使用 GraphQL 来查询图像数据 37。`gatsbyImageData` 解析器是实现所有优化的关键。

#### 实施指南：使用 `GatsbyImage` 的动态博客文章

1. **配置：** 安装 `gatsby-plugin-image`、`gatsby-plugin-sharp`、`gatsby-transformer-sharp` 23。同时配置 `gatsby-source-filesystem` 以引入本地 Markdown 文件或连接到 CMS 43。

2. **GraphQL 查询（在页面模板或 `gatsby-node.js` 中）：**
    ```graphql
    query BlogPostQuery($slug: String!) {
      mdx(frontmatter: { slug: { eq: $slug } }) {
        frontmatter {
          title
          author
          hero_image {
            childImageSharp {
              gatsbyImageData(
                width: 800
                placeholder: BLURRED
                formats:
              )
            }
          }
        }
      }
    }
    ```

    (示例改编自 39)。

    在此查询中，placeholder: BLURRED 指示 Gatsby 在构建时生成模糊的 Base64 占位符。其他选项包括 DOMINANT_COLOR 或 TRACED_SVG。

3. **React 组件：**

    ```javascript
    import { graphql } from 'gatsby'
    import { GatsbyImage, getImage } from 'gatsby-plugin-image'

    function BlogPostTemplate({ data }) {
      const image = getImage(data.mdx.frontmatter.hero_image)

      return (
        <section>
          <h1>{data.mdx.frontmatter.title}</h1>
          <GatsbyImage image={image} alt={data.mdx.frontmatter.title} />
          {/*... rest of post */}
        </section>
      )
    }
    export const query =... // (GraphQL query from above)
    ```

    (示例改编自 39)。

    getImage 是一个辅助函数，用于从查询结果中提取图像数据。最终传递给 <GatsbyImage> 的 image 对象包含了所有 srcSet、sizes 以及构建时生成的 Base64 占位符。


### C. 战略比较：`next/image` (按需) vs. `gatsby-plugin-image` (构建时)

Gatsby 在构建时处理所有图像 38，而 Next.js 按需处理它们 19。这种差异导致了最核心的架构权衡：

- **Gatsby (构建时支付成本)：**

    - **模型：** 静态完美。Gatsby 会在构建时处理每一张图片，生成所有变体和占位符 46。

    - **优势：** 网站是 100% 静态优化的。即使用户是第一个访问者，也能获得最快的 LCP 和即时的占位符 47。

    - **代价：** 构建时间。对于拥有数万张图片的大型网站（如电商网站），Gatsby 的构建时间可能会变得非常长，甚至无法接受 38。

- **Next.js (运行时支付成本)：**

    - **模型：** 动态可扩展。构建速度非常快 47，因为它跳过了对远程图像的处理。

    - **优势：** 即使有数百万张图片，构建也能在几秒钟内完成。非常适合动态内容和用户生成内容 (UCG) 47。

    - **代价：** “首次加载”的轻微延迟。访问特定图像的 _第一个_ 用户会触发图像的动态生成。这个请求会稍慢一些。但一旦生成，结果就会在 CDN 上被永久缓存，所有后续访问者都会获得极快的速度。


**选择的关键不在于“哪个更快”，而在于“你希望在何时支付优化的成本”。**

#### 架构对比：`next/image` vs. `gatsby-plugin-image`

|**特性**|**next/image (Next.js)**|**gatsby-plugin-image (Gatsby)**|
|---|---|---|
|**优化策略**|**按需（运行时）** 通过服务器/边缘节点 [19, 38]|**构建时** 通过 Node.js 37|
|**数据获取**|简单的 `src` URL 字符串 [24, 38]|**GraphQL 查询** (`gatsbyImageData`) 37|
|**占位符生成**|本地 `import` 图像：**自动** 20。<br><br>  <br><br>远程 `src` 图像：**手动** (`blurDataURL`) 28|在 GraphQL 查询中 **可配置** (`placeholder: BLURRED`) [39]|
|**构建时间影响**|**最小**。与图像数量无关 47。|**高**。随图像数量线性增长 38。|
|**远程图像支持**|原生且灵活 [19, 38]|必须在 GraphQL 中配置数据源 [38, 39]|
|**现代格式**|WebP, AVIF (自动) 19|WebP, AVIF (可在 GraphQL 中配置) [18, 39]|
|**理想用例**|动态网站、电商、UCG、需要快速构建 47|静态网站、博客、作品集、追求极致性能 47|

## 第三部分：手动实现与纯 React 库

在某些情况下，开发者可能无法使用上述框架，例如在遗留的 React 项目（如 Create React App）中，或使用 Vite 的纯 React 应用，或者需要对加载逻辑进行完全的自定义控制时。

### A. 浏览器原生延迟加载：最简单的方法

最简单的延迟加载形式是使用 `<img>` 元素上的 `loading="lazy"` 属性 4。

- **React 实现：**

    ```html
    <img src={imageUrl} loading="lazy" alt="decription" width="600" height="400" />
    ```

    (示例改编自 50)。

- **分析：**

    - **优点：** 极其简单，零库成本，由浏览器原生支持 4。

    - **缺点：** 无法提供自定义的 "blur-up" 占位符体验 51。它只会使用浏览器默认的占位符（或一片空白），这在视觉上可能仍然很突兀。

    - **关于 React 的误区：** 有一种误解认为，由于 React 是动态构建 DOM 的，`loading="lazy"` 可能不会生效 52。这是不正确的。该属性指示浏览器推迟 _网络请求_，而不是 DOM 插入。即使 React 已经渲染了 `<img>` 标签，只要它在视口之外，浏览器就不会下载 `src`。


#### 浏览器支持：`loading="lazy"`

该属性的兼容性在过去是一个问题，但现在已获得广泛支持，使其在生产环境中完全可用。

|**浏览器**|**版本支持**|**备注**|
|---|---|---|
|Chrome|✅ (77+)|支持|
|Firefox|✅ (75+)|支持|
|Safari|✅ (15.4+)|**关键点：** 曾是最后的支持者，现已支持 53|
|Edge|✅ (79+)|支持|
|**全球**|**~94%**|**已准备好用于生产** 53|

(数据来源于 53)

### B. Intersection Observer API：现代手动方法

`IntersectionObserver` API 是现代浏览器中用于检测元素可见性的首选方式 3。它完全异步，不会像 `scroll` 监听器那样阻塞主线程 5。

开发者可以构建一个自定义的 React Hook (`useIntersectionObserver` 或 `useFirstViewportEntry`) 来封装这个 API 54。

- **Hook 逻辑（概念）：**

    1. 使用 `useRef` 来引用需要观察的 DOM 元素 56。

    2. 使用 `useState` 来跟踪元素是否已进入视口 (`isIntersecting`) 54。

    3. 使用 `useEffect` 来创建 `IntersectionObserver` 实例，并调用 `observe()` 开始观察 `ref.current`。

    4. 在 `useEffect` 的清理函数中，调用 `disconnect()` 来停止观察，以防止内存泄漏 54。

    5. 一旦元素相交，更新状态，并可以选择断开观察（如果只需要触发一次）54。

- **组件使用：**

    ```jsx
    const [ref, isIntersecting] = useIntersectionObserver({ threshold: 0.1 });

    return (
      <div ref={ref}>
        {isIntersecting? <ActualImageComponent src={...} /> : <Placeholder />}
      </div>
    );
    ```


### C. 第三方库：抽象化 Intersection Observer

对于大多数纯 React 项目，从头开始编写 `IntersectionObserver` 逻辑是不必要的。许多库已经很好地封装了这一功能，并添加了关键的 _占位符_ 支持 57。

- **库对比：**

    - `react-lazyload`：一个流行且成熟的库，但它可能使用较旧的事件监听器技术（尽管它经过了高度优化）57。

    - `react-lazy-load-image-component`：一个更现代的选择，它明确使用 `IntersectionObserver`，并且原生支持 "blur-up" 等占位符效果，使其成为 LQIP 的理想选择 57。


#### 实施指南：`react-lazy-load-image-component` (Blur-Up)

这是在纯 React 中实现 "blur-up" 效果的推荐方法，因为它提供了最佳的 UX 与开发工作的比率。

1. **安装：** `npm install --save react-lazy-load-image-component` 58。

2. **准备占位符：** 你需要自己创建低分辨率的图像（例如，一个 20px 宽的.jpg）58。

3. **实现：**

    ```jsx
    import React from 'react';
    import { LazyLoadImage } from 'react-lazy-load-image-component';
    import 'react-lazy-load-image-component/src/effects/blur.css';

    // highResImage: 'path/to/full-image.jpg'
    // lowResPlaceholder: 'path/to/tiny-placeholder.jpg'

    const MyLazyImage = ({ highResImage, lowResPlaceholder }) => (
      <LazyLoadImage
        alt="A description of the image"
        src={highResImage}
        placeholderSrc={lowResPlaceholder}
        effect="blur"
        width={600} // 必须提供尺寸以防止 CLS
        height={400} // 必须提供尺寸以防止 CLS
      />
    );
    ```

    (代码示例基于 58)。

    关键在于 placeholderSrc 属性，它指定了低质量的图像，而 effect="blur" 则应用了模糊和淡入的过渡效果。


### D. 手动 LQIP "Blur-Up" 技术 (DIY)

如果不想添加库，开发者可以手动复制 `react-lazy-load-image-component` 的核心逻辑。

1. useProgressiveImg Hook：

    这是一个自定义 Hook，用于封装“从低分辨率切换到高分辨率”的加载逻辑 59。

    ```jsx
    import React from 'react';

    const useProgressiveImg = (lowQualitySrc, highQualitySrc) => {
      const = React.useState(lowQualitySrc);

      React.useEffect(() => {
        setSrc(lowQualitySrc);

        const img = new Image();
        img.src = highQualitySrc;

        img.onload = () => {
          setSrc(highQualitySrc);
        };
      },);

      return;
    };

    export default useProgressiveImg;
    ```

    (代码改编自 59)。

    这个 Hook 始终返回一个 src。它首先返回低质量的 src，同时在后台加载高质量的 src。加载完成后，它会更新状态，返回高质量的 src。它还方便地返回一个布尔值，告诉我们当前是否应应用模糊效果。

2. **组件与 CSS：**

    ```jsx
    import useProgressiveImg from './useProgressiveImg';

    // 必须在 CSS 文件中定义
    //.progressive-image {
    //   filter: blur(20px);
    //   transition: filter 0.3s ease-out;
    // }
    //.progressive-image-loaded {
    //   filter: blur(0);
    // }

    const MyImageComponent = ({ lowRes, highRes, alt }) => {
      const [src, { blur }] = useProgressiveImg(lowRes, highRes);

      return (
        <img
          src={src}
          alt={alt}
          className={`progressive-image ${blur? '' : 'progressive-image-loaded'}`}
          style={{ width: '100%' }} // 确保容器已设置尺寸以防 CLS
        />
      );
    };
    ```

    (实现思路源自 10)。

    这种方法的关键是 CSS transition 属性 60。当 progressive-image-loaded 类被添加时，filter 属性会平滑地从 blur(20px) 过渡到 blur(0)，从而创建出优雅的“解模糊”动画 61。


## 第四部分：高级占位符策略：BlurHash vs. 渐进式 JPEG

最后，我们深入探讨占位符本身的性质。我们之前讨论的 LQIP（Base64 或微小 JPEG）、BlurHash 和渐进式 JPEG 在技术和性能上有着根本的不同。

- **LQIP (Base64)：** 这是 `plaiceholder` 和 `gatsby-plugin-image` 生成的。它是一个 _真实的_ 图像文件，被编码为 `data:image/...;base64,...` 字符串 29。

    - **优点：** 浏览器原生渲染，零客户端解码成本。

    - **缺点：** Base64 字符串比原始二进制文件大约 30%，但对于一个 2kB 的缩略图来说，这微不足道。

- **BlurHash：** 这是一种 _算法字符串表示_ 11。

    - **优点：** 负载极小（约 30 字节）12，非常适合嵌入到 API 的 JSON 响应中 63。

    - **缺点：** _不是_ 图像。它需要在客户端使用 JavaScript 库（如 `react-blurhash`）进行 _解码_，这会增加 bundle 大小和客户端 CPU 开销 64。

- **渐进式 JPEG：** 这是一个 _文件格式_ 16。


**结论：** 对于 Web 开发，**LQIP (Base64)** 通常是最佳选择，因为它提供了良好的压缩和原生的浏览器渲染。**BlurHash** 更适合于移动原生应用或数据负载极其敏感的 API 驱动场景。

### A. BlurHash：紧凑的字符串占位符

BlurHash 是一种将图像的模糊预览编码为短字符串的算法 11。

- 服务器端实施 (Node.js)：

    这是一个两步过程 67。

    1. 使用 `sharp` 加载图像，将其调整到非常小（例如 32px 宽），并提取原始的 `rgba` 像素数据 67。

    2. 将原始像素数据和尺寸传递给 `blurhash` 库的 `encode` 函数以生成哈希字符串 67。

- **客户端实施 (React)：**

    1. 安装 `react-blurhash` 68。

    2. 将从 API 获取的哈希字符串传递给 `<Blurhash>` 组件。

    ```jsx
    import React, { useState, useEffect } from 'react';
    import { Blurhash } from 'react-blurhash';

    const ImageWithBlurHash = ({ hash, src, alt }) => {
      const [isLoaded, setLoaded] = useState(false);
      const = useState(null);

      useEffect(() => {
        const img = new Image();
        img.src = src;
        img.onload = () => {
          setLoaded(true);
          setImgSrc(src);
        };
      }, [src]);

      return (
        <div style={{ position: 'relative', width: 400, height: 300 }}>
          {!isLoaded && (
            <Blurhash
              hash={hash} // e.g., "LKO2?U%2Tw=w]~RBVZRi};RPxuwH"
              width={400}
              height={300}
              resolutionX={32}
              resolutionY={32}
              style={{ position: 'absolute', top: 0, left: 0 }}
            />
          )}
          <img
            src={imgSrc}
            alt={alt}
            style={{
              width: '100%',
              height: '100%',
              opacity: isLoaded? 1 : 0,
              transition: 'opacity 0.3s'
            }}
          />
        </div>
      );
    };
    ```

    (组件示例改编自 68)。


### B. 渐进式 JPEG：基于格式的解决方案

我们再次审视基线 JPEG 和渐进式 JPEG 之间的视觉差异。基线 JPEG 是从上到下加载的，而渐进式 JPEG 是通过多次扫描从模糊到清晰加载的 13。

- 关键分析：

    必须明确，这 不是 一种 React 技术，而是一种资产编码的选择 16。渐进式 JPEG 在十年前是一个巨大的进步，因为它提供了即时的（尽管是模糊的）反馈。

- 现代观点：

    如今，这种技术在很大程度上已被“延迟加载 + LQIP”模式所取代 1。LQIP 模式提供了更平滑、更可控的视觉过渡（例如，CSS filter 动画），并且不会像渐进式 JPEG 那样在解码时给客户端带来额外的（尽管很小）计算开销 15。现代图像格式 WebP 和 AVIF 也有它们自己的渐进式渲染模式，但 next/image 和 gatsby-plugin-image 等工具通过它们的 LQIP 策略使这种格式层面的渐进式渲染变得不那么必要了。

- 如何创建：

    如果确实需要，可以使用 Adobe Photoshop (在“存储为 Web 所用格式”中勾选“渐进式”) 70 或 Optimizilla 等在线工具 72 来创建渐进式 JPEG。


## 第五部分：最终建议与战略综合

综合本报告的分析，理想的图像加载策略是一个多层组合：

1. **何时 (When)：** 使用 **延迟加载** (Lazy Loading) 来推迟视口外图像的加载。

2. **期间 (During)：** 使用 **LQIP**（低质量图像占位符，通常是 Base64 "Blur-Up"）来立即填充空间、防止 CLS 并改善感知性能。

3. **如何 (How)：** 最终提供的图像应采用 **现代格式**（如 WebP 或 AVIF）并进行 **响应式调整**（`srcset`），以确保为特定设备提供最小的文件大小。


基于您的项目场景，推荐的实施路径如下：

### 场景一：新 React 项目（绿地项目）

- **推荐：** 使用 **Next.js** (`next/image`)。

- **理由：** Next.js 提供了无与伦比的灵活性。其按需优化模型可实现快速构建，并可无限扩展到大型、动态的网站（如电商）20。

- **实施：**

    - 对于本地静态图像（如 Logo、页面横幅）：使用 `import` 导入图像，`placeholder="blur"` 将自动生效 20。

    - 对于来自 CMS 的远程图像：在 `getStaticProps` 或 `getServerSideProps` 中使用 `plaiceholder` 库 29 或“嵌套优化器”技巧 33 来动态生成 `blurDataURL`。


### 场景二：静态博客或作品集（媒体集有限）

- **推荐：** **Gatsby** (`gatsby-plugin-image`)。

- **理由：** 如果构建时间不是主要障碍，Gatsby 的构建时优化方法将生成一个“完美”的静态网站 47。所有占位符和响应式尺寸都已预先生成，可从 CDN 快速提供，无需任何运行时计算。

- **实施：** 遵循本报告第二部分 B 节中的 GraphQL `gatsbyImageData` 工作流 39。


### 场景三：现有的纯 React 应用（遗留项目 / CRA / Vite）

- **推荐：** 采用分层方法。

- **级别 1 (最简单)：** 对于非核心图像（例如页脚图标），立即在整个代码库中添加 `loading="lazy"` 属性 50。它不需要库，并且拥有出色的浏览器支持 53。

- **级别 2 (最佳 UX)：** 对于面向用户的关键图像（如产品画廊、文章图片），使用 `react-lazy-load-image-component` 库。它的 `effect="blur"` 属性是实现高质量 "blur-up" 体验的最快、最简单的方法 58。

- **级别 3 (完全控制)：** 如果您极度关注 bundle 大小并希望避免任何第三方库，请手动实现。结合使用自定义的 `useIntersectionObserver` Hook 54 和 `useProgressiveImg` Hook 59，并通过 CSS `filter` 过渡 10 来实现您自己的 LQIP "blur-up" 效果。
