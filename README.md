# TSKaigi 2024 “思考 Prettier 的未来” 演讲笔记

发布于 2024/05/11

这是在 TSKaigi 2024 上的发表演讲“思考 Prettier 的未来”的演讲笔记。演示幻灯片内容较少，将在演讲后发布。

---

大家好，今天我想和大家讨论的主题是“思考 Prettier 的未来”。

我叫鈴木颯介，在 Ubii 公司担任产品开发工程师的同时，在筑波大学学习计算机。我喜欢开源软件，今天将要讨论的 Prettier 就是我负责维护的项目之一，我也是转译器 Babel 的贡献者，并且最近向 WebKit 的 JS 引擎提交了很多补丁。

我所在的 Ubii 公司是 TSKaigi 的金牌赞助商。我们有一个展位，在那里除了 Ubii 的商品外，还提供 Prettier 的贴纸，如果您感兴趣，请一定要来看看。

首先，可能有些人对 Prettier 不太熟悉，让我简单介绍一下。Prettier 是一个支持 JavaScript、TypeScript、HTML、CSS 等语言的代码格式化工具。其最大的特点是配置项少，而且格式化的结果不因编写者不同而有所不同，这是一种所谓的“有观点”的代码格式化工具。

可能有人已经知道了，但让我介绍一个具体的例子。左侧的代码是应用 Prettier 前的样子，右侧的代码是应用 Prettier 后的样子。这种转换是自动完成的。

关于编码风格的立场往往受到个人经验和价值观的影响。在代码审查时，可能会因为“我喜欢这种风格”或“按我的经验，这种风格会有这样的问题”等不太本质的讨论而浪费时间，每个人可能都经历过这样的情况。Prettier 的目的就是消除这种关于编码风格的讨论和争议。

像 Go 和 Rust 这样的一些编程语言，其编译器提供者也提供了类似 Prettier 这样有观点的代码格式化工具。

Prettier 由 James Long 在 2017 年开发，并作为开源软件发布。

说到 2017 年，React 已经普及了，但 James Long 当时没有使用 JSX 来编写 React。他使用的 Emacs 上没有好用的 JSX 代码格式化工具，他需要自己调整缩进和空格。当时他也在使用 Facebook 开发的 ML 系语言 Rescript。Rescript 附带了一个名为 refmt 的代码格式化工具，提供了与今天的 Prettier 类似的体验，James Long 非常喜欢它。受到这种体验的启发，他开发了 Prettier。

之后，Facebook 的工程师 Vjeux 作为工作的一部分，全职参与了 Prettier 的开发并提升了其质量。现在，Prettier 已经确立了在 JavaScript 生态系统中类似事实标准的地位。

尽管 James Long 和 Vjeux 后来不再参与开发，但 Prettier 的维护工作一直由一个活跃的志愿者团队负责。

顺便说一下，我是从 2019 年开始参与 Prettier 的开发的，那时它已经支持了现在的语言，并且使用感和现在差不多。我是其中一个维护时间最长的维护者之一，但我不是现在的 Prettier 团队的原始成员。

自 Prettier 诞生以来已经过去了约 7 年。由于长期以来有很多用户使用，一些问题已经变得明显。其中最重要的是使用不便和执行速度慢的问题。

首先，关于使用不便的问题。实际上，如果单独使用 Prettier，使用上应该不会太麻烦。或者说，因为几乎不需要配置，所以应该很容易使用。

但实际上，Prettier 通常与 ESLint 等其他工具一起使用。Prettier 是一个代码格式化工具，而不是一个 linter。它会自动修正编码风格，但不涉及更深层的问题。例如，对于从未重新赋值的变量使用 let 而不是 const 等问题，不是 Prettier 的责任。因此，为了自动检测这类问题，需要使用其他工具。目前，ESLint 可能是最常使用的工具。

将 Prettier 和 ESLint 结合使用时的配置非常繁琐，使用不便，这是常听到的抱怨。ESLint 是一个 linter，但也包含一些像代码格式化工具一样的规则。虽然这些风格相关的规则最近已经从 ESLint 的核心中移除了，但以前如果使用 ESLint 推荐的规则集，这些风格规则就会生效，并与 Prettier 的格式化结果发生冲突，这是一个常见的问题。

为了避免这个问题，ESLint 和 Prettier 的用户需要安装 eslint-config-prettier 或 eslint-plugin-prettier 等，并进行相应的配置。这些配置一旦习惯了并不难，但对于只想简单结合使用 ESLint 和 Prettier 的新用户来说，这确实是不便利的。

此外，这不是 Prettier 的问题，但 ESLint 不支持 TypeScript。因此，如果想使用 ESLint 和 Prettier 以及 TypeScript，至少需要安装 eslint、prettier、@typescript-eslint/parser、@typescript-eslint/eslint-plugin、eslint-config-prettier、typescript 等包。这其实相当麻烦。

接下来是执行速度慢的问题。这取决于如何使用 Prettier，具体问题和原因各不相同。

首先，考虑在 VSCode 等文本编辑器的扩展中，在保存时执行格式化的使用场景。在这种情况下，大多数情况下性能不会成问题。但如果文件大小太大，你会明显感觉到 Prettier 的执行需要较长时间。这是相当有压力的。

再考虑在 CI 或本地 CLI 中对整个项目执行 Prettier 的情况。这是相当慢的。不必要地慢。当我们在 Ubii 的代码库上执行 Prettier 时，我感到了很大的压力。

正如您所见，当前的 Prettier 存在一些问题。在过去几年中，一些解决这些问题的工具已经被开发出来并变得有名。

首先，作为一个解决这些问题的例子，让我们考虑一下 Deno。Deno 是 Node.js 的创建者 Ryan Dahl 基于他在 Node.js 中的经验和失败，开发的一个新的 JavaScript 运行时。与 Node.js 不同，Deno 默认可以执行 TypeScript，并且需要明确授权才能访问文件和网络，使用基于 ECMAScript Modules 的模块系统，改善了 Node.js 的一些细微之处。此外，只需运行 deno lint 或 deno fmt 命令，就可以无需配置地执行 linter 和 formatter，这是一个特点。

deno lint 命令执行的是由 Rust 编写的自定义 linter deno_lint，而 deno fmt 命令执行的是 Deno 团队的成员开发的同样由 Rust 编写的 formatter dprint。这意味着使用 Deno 的人不会遇到前面讨论的 Prettier 的问题。

我个人认为这是一种非常好的体验。我最近在制作一些简单的 CLI 工具时会使用 Deno，而无需配置就可以使用 TypeScript、linter 和 formatter，这让我感觉非常轻松。我认为自己对 linter 和 formatter 相对熟悉，但即使是这样，我也觉得这很方便。对于不太熟悉这些工具的人来说，体验会更好。当然，Deno 并不是仅为了解决 linter 和 formatter 的问题而创建的，但作为解决这些问题的方式，它确实非常聪明。

如果继续使用 Node.js，一个好的工具是 Biome。Biome 的诞生有些复杂。Biome 是作为另一个工具 Rome 的社区分支而诞生的。Rome 是由 Babel 和 Yarn 的创建者 Sebastian McKenzie 开发的。

Rome 的目标是前端工具链的大统一。为了进行现代前端开发，需要掌握多个工具。首先是 Babel 或 SWC 这样的转译器，webpack 或 Rollup 这样的模块打包工具，Terser 这样的缩小器，ESLint 这样的 linter，Prettier 这样的 formatter，Jest 这样的测试运行器，等等。最近，像 Next.js 这样的大型元框架已经吸收了这方面的一些复杂性，可能不需要直接关注这些问题，但这些工具在内部仍然是分开运行的。

这种情况对于需要设置多个工具的程序员来说是一种负担，从计算资源的角度来看，也是一种浪费。因为这些工具虽然在某些路径上做相同的事情，但不共享结果。例如，模块打包工具和转译器以及 linter 和 formatter 都会解析 JavaScript 代码，但解析出的 AST 并不共享，同一代码会被不同工具多次解析。

为了解决这些问题，Rome 应运而生。Rome 的创建者 Sebastian 成立了一家公司并推进开发，但公司最终解散了。具体情况我不清楚，但毕竟是创业，可能会有各种情况。

尽管 Rome 公司已经解散，但当时已经开发出了一些可用的 linter 和 formatter。因此，Rome 的成员 ematipico 将 Rome 分支出来，并启动了 Biome 项目。我不确定 Biome 是否已经放弃了统一前端工具链的大梦想，但至少目前看来，它主要专注于开发 linter 和 formatter。

Biome 的一个最大特点是，它同时具备了 linter 和 formatter，并且执行速度非常快。这两点就解决了 Prettier 的两大问题。开始使用 Biome 只需要运行以下命令：

```shell
npm install --save-dev --save-exact @biomejs/biome
npx @biomejs/biome init
```

我经常被问到“你怎么看待 Biome？”这个问题，所以借此机会我会明确地说，我喜欢 Biome。我认为这是一个非常棒的项目，我支持它。

虽然我个人喜欢 Biome，但作为 Prettier 的维护者，我必须考虑这些工具的存在以及 Prettier 的未来应该是怎样的。现在来谈谈正题。

比较相同用途的软件时，通常会使用一张每个功能作为一行的表格。这种表格在用来宣传自己的软件时可能会变得有所偏颇，但在真正想比较软件性质时，它是一个很有用的工具。这里我想制作一张比较 Prettier 和 Biome 的表格。

<img src="https://storage.googleapis.com/zenn-user-upload/395af0137bfc-20240511.png">

让我们看看这些行。

首先是算法。Biome 和 Prettier 在这方面完全相同。它们使用的中间表示也相同，兼容性很高。就 JavaScript 而言，兼容性达到 97%。以前它们的行为就非常相似，但通过 Prettier Challenge，兼容性得到了进一步提高。Prettier Challenge 是 Prettier 组织的一次活动，Prettier 向开发了与 Prettier 兼容的 Rust 制 JavaScript 格式化工具的组织捐赠了 20,000 美元。Biome 为了这次活动提高了与 Prettier 的兼容性，并因此获得了奖金。

接下来是格式化单个文件时的性能。Biome 很快，Prettier 虽然不是太慢，但与 Biome 相比也不快。Prettier 希望能追上 Biome，但说实话这很难。Prettier 格式化单个文件的执行已经足够优化，但要大幅加快速度是困难的。Prettier 格式化文件所花的时间大约一半是解析器，而解析器是使用外部库的。即使假设打印处理足够快，要加倍速度也很难，这种程度远远达不到 Biome 的水平。

接下来是格式化多个文件时的性能。Biome 很快。Prettier 在一般的 Web 公司的代码库中感觉到压力。Prettier 可能很难赶上 Biome，但我认为有可能显著改善 Prettier。Prettier 的缺点在这种情况下也是优点，因为之前没有人试图加快速度。这意味着还有很大的性能改善空间。Prettier 的 CLI 执行缓慢不是因为 CPU 密集，而是因为 IO 密集。一位名叫 Fabio 的工程师正着眼于这一点，正在进行一项将 Prettier 的 CLI 完全重写的实验。目前，对 Babel 存储库的执行结果比以前快了 10 倍以上。这个新的 CLI 计划在下一个主要版本 v4 中推出。但即使快了 10 倍，也还远远赶不上 Biome。

接下来是与某个 linter 的集成。Biome 自带 linter，所以这很简单。Prettier 不论与哪种 linter 集成，都有些麻烦。这是 Biome 的一个很好的点。从 Twitter 上 Biome 用户的声音来看，体感上，简单的设置比性能更受欢迎。Prettier 要接近这种体验，可能需要准备一些封装，或者提供一些设置用的脚手架命令。

接下来是与 ESLint 的集成。这在两者中都有些麻烦。我个人认为这非常重要。ESLint 是一个可以非常灵活设置的 linter。如果你从使用 ESLint 和 Prettier 的环境转移到 Biome，转移 formatter 部分可能很简单，但 linter 部分会很困难。特别是如果你熟练使用 ESLint，那么复制 ESLint 的行为到 Biome Linter 可能是不可能的。我所在的 Ubii、Prettier、Babel 都为自己的代码库制定了定制规则并进行了操作。这种情况下，定制规则很有效。此外，ESLint 社区维护的各种插件中包含的规则在 Biome Linter 中也不存在。因此，实际上即使可以剥离 Prettier，也不能剥离 ESLint 的情况存在，如果 Biome Formatter 和 ESLint 同时使用，最终配置麻烦，满意度会减半。我不知道 Biome 是否在计划解决这个问题，但如果能做些什么，可能需要制作封装或精心设计脚手架。

接下来是当前支持的语言。Biome 支持 TypeScript、JavaScript、JSX、TSX、JSON、JSONC。而 Prettier 除此之外还默认支持 Flow、HTML、Vue、Angular、Handlebars、CSS、Less、SCSS、SaSS、GraphQL、Markdown、YAML，并有为 Ruby、Astro、Svelte 维护良好的插件。目前，Prettier 支持的语言更多。Prettier 使用现有的高质量解析器库来支持新语言，因此比较容易支持新语言。例如，作为 JavaScript 的解析器，它使用 Babel，作为 TypeScript 的解析器，它使用与 typescript-eslint 相同的东西。而 Biome 是自己编写并维护解析器的。我认为这是因为它们希望尽可能减少依赖，并使用支持 Red Green Tree 的解析器。支持新语言较慢可能是因为这个原因。

接下来是插件。Biome 目前似乎正在计划如何实现插件。维护者计划使用名为 GritQL 的 DSL 构建插件系统，部分实现已在进行中。Prettier 允许用 JavaScript 编写插件。Prettier 的插件本来是为了支持新的语言而设计的，但像 prettier-plugin-tailwindcss 这样的插件也用于改变已支持语言的格式化行为。这种用法实际上相当棘手，作为 Prettier 的维护者，这让我有些矛盾的感觉，但实际上它被广泛使用。如果 Biome 能够实现一个与 Prettier 相似、可以用 TypeScript 编写而不降低性能的插件系统，那将是真正了不起的。我很好奇它将采取何种形式。这将是决定 Biome 生态系统未来走向的重要决策。

最后是各自软件的编写语言。Biome 用 Rust 编写，Prettier 用 JavaScript 编写。这经常被讨论，但通常 Rust 更容易编写快速程序。而另一方面，作为一种动态编程语言，JavaScript 具有高度的灵活性，构建插件系统等相对容易。这些语言的特点清楚地体现在 Biome 和 Prettier 的特点中。

关于 Rust 和 JavaScript 的差异，最终分发的形式也不同。例如，Biome 虽然通过 npm 分发，但最终得到的是二进制文件。而 Prettier 则是得到 JavaScript 文件。这是一个重大差异，但实际上可能不那么重要。因为如果 Prettier 与 Node.js 一起打包，就可以制成二进制文件，而如果 Biome 编译成 WebAssembly，只要有 Node.js 就可以运行。目前，双方都还不支持这种形式，但如果愿意，是可以做到的。

最后，哪种分发形式更好取决于情况。如果是 JavaScript，只要机器上安装了 Node.js，就可以在不同的 OS 或 CPU 上运行。反过来说，这需要 Node.js。如果是二进制分发，即使没有 Node.js 也可以运行，但需要安装适合自己环境的二进制文件。

现在，让我们重新整理一下 Prettier 今后能提供的价值和应该关注的点。

首先，Prettier 今后能提供的价值在于支持的语言种类多。Prettier 可以利用其他 JavaScript 生态系统工具中使用的高质量解析器，这使得 Prettier 团队无需支付大量维护成本，就可以稳定支持多种语言。

此外，由于是用 JavaScript 编写的，灵活的插件系统也是 Prettier 的优势。虽然目前如果不使用一些技巧，无法改变现有语言的格式，但这方面还有发展的余地，例如官方支持。

另一方面，应该关注的点是便利性和性能。关于便利性，主要是加强与 ESLint 的集成。由于现在的 ESLint 推荐规则集中不再包含风格相关的规则，因此通常不需要 eslint-config-prettier，实际上已经相当便利，但还需要更清晰的指导。

最难的问题还是性能。老实说，实现与 Biome 相当的性能非常困难。但 Prettier 还是需要尽力而为。首先是发布新的主要版本 v4，装载了 Fabio 开发的新 Prettier CLI。然后，我们的代码库可能不会改写为 Rust 或 Go。即使要做，也将是另一个项目。

今天，我讨论了 Prettier 的诞生、与强大的竞争对手 Biome 的比较以及 Prettier 的未来。到目前为止，Prettier 没有强大的竞争对手，我们忽视了自我提升的努力。Biome 的出现使我们双方都变得更强，结果改善了 TypeScript/JavaScript 代码格式化工具的质量，提升了用户的开发体验。

演讲到此结束。谢谢大家。

演示幻灯片中的信息很少，我将在稍后发布演讲笔记。
