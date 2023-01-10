---
title: 解析unocss引擎的工作原理
date: 2023-01-10T22:00:00.000+00:00
lang: zh
duration: 20min
---
在此之前，你首先要知道什么是[unocss](https://github.com/unocss/unocss)

## 你能学到什么?
- [x] 属于你自己的原子化css引擎

我的代码将会提交到这个[github仓库](https://github.com/SnowingFox/mini-unocss)

## 预备知识
这些东西都是构建一个原子化引擎必不可少的内容，也是unocss的所有特性，我们将一一实现它们，不过在此之前你应该知道怎么去用他们
- [Custom Rules](https://github.com/unocss/unocss#custom-rules)
- [Shortcuts](https://github.com/unocss/unocss#shortcuts)
- [Preflight](https://github.com/unocss/unocss#preflight)
- [Custom variants](https://github.com/unocss/unocss#custom-variants)
- [Theme](https://github.com/unocss/unocss#extend-theme)
- [Layers](https://github.com/unocss/unocss#layers)
- [Utilities Preprocess & Prefixing](https://github.com/unocss/unocss#utilities-preprocess--prefixing)

我们将围绕上述功能来实现一个mini-unocss

## 工作流程

在了解此之前我希望你能读完unocss作者的构思unocss的一篇博客https://antfu.me/posts/reimagine-atomic-css-zh

[@unocss/core](https://github.com/unocss/unocss/tree/main/packages/core)是一个css原子化引擎，它并不包含任何预设

举个最简单的例子
```ts
const fixture = '<div class="text-red">hello</div>'

const uno = creaeteGenerator({
  rules: [
    ['text-red', { color: 'red' }]
  ]
})

const { css } = uno.generate(fixture)
```

css的内容
```css
/* layer: default */
.text-red{color:red;}
```
首先我们需要`extractor`(提取器)将下列文本进行解析提取
```tsx
<div className="text-red">hello</div>
```
可以发现最后提取出我们自定义的规则`text-red`，具体做法就是用正则匹配，后面会说到，这里不作赘述

我们把提取出来的`text-red`叫做`token`，而后我们会解析一遍这个token所包含的信息，发现它有如下规则
```ts
['text-red', { color: 'red' }]
```

最后根据这个规则生成一个css文本内容
```
.text-red{color:red;}
```

其实原理就是这么简单，很多框架的原理都很简单，跟着我的脚步，到了最后你也能写出一个原子化css引擎

## extractor tokens

假设我们现在有如下rule
```ts
rules: [
  ['text-red', { color: 'red' }]
]
```
而我们需要解析的文本如下
```tsx
<div className="text-red" />
```
那我们就需要用到正则匹配，所以我们需要一个解析这种规则的工具函数

```ts
const validateFilterRE = /[\w\u00A0-\uFFFF-_:%-?]/

function isValidSelector(selector = ''): selector is string {
  return validateFilterRE.test(selector)
}

const defaultSplitRE = /\\?[\s'"`;{}]+/g
const validateFilterRE = /[\w\u00A0-\uFFFF-_:%-?]/

function isValidSelector(selector = '') {
  return validateFilterRE.test(selector)
}
const arbitraryPropertyCandidateRE = new RegExp(`^${arbitraryPropertyRE.source}$`)
const splitCode = (code) => {
  const result = new Set()

  for (const match of code.matchAll(arbitraryPropertyRE)) {
    if (!code[match.index - 1]?.match(/^[\s'"`]/))
      continue

    result.add(match[0])
  }

  code.split(defaultSplitRE).forEach((match) => {
    isValidSelector(match) && !arbitraryPropertyCandidateRE.test(match) && result.add(match)
  })

  return [...result]
}
```
解析上诉文本后可得
```
['<div', 'class=', 'text-red', '/>']
```

为了预防某些token没被解析到，我们可以暴露一个`safelist`的配置，确保safelist内配置好的token能够在运行时用到

```ts
const tokens = await this.applyExtractors(input, options?.id)
// safelist
this.config.safelist!.forEach(s => tokens.add(s))
```

目前我们的代码如下

```ts
export class UnoGenerator<Theme extends {} = {}> {
  private _cache = new Map<string, any>()
  public config: UserConfig<Theme>
  public blocked = new Set<string>()

  constructor(public userConfig: UserConfig<Theme>) {
    this.config = resolveConfig(userConfig)
  }

  async generate(input: string, options?: GenerateOptions) {
    const tokens = await this.applyExtractors(input, options?.id)
  }

  async applyExtractors(input: string, id?: string) {
    const tokenSet = new Set<string>()

    const extractContext: ExtractorContext = {
      original: input,
      id,
      code: input,
    }

    if (this.config.extractors) {
      for (const extractor of this.config.extractors) {
        const extractedTokens = await extractor.extract(extractContext)

        if (extractedTokens) {
          for (const token of extractedTokens)
            tokenSet.add(token)

        }
      }
    }

    return tokenSet
  }
}
```

## parse tokens
ok, 接下来就是解析token了，先放源码
```ts
const layerSet = new Set<string>([DEFAULT_LAYER])
const sheet = new Map<string, SheetUtils>()

const tokenPromises = Array.from(tokens).map(async(raw) => {
  if (matched.has(raw) || this.isBlocked(raw))
    return

  const payload = await this.parseToken(raw)

  if (!payload)
    return

  matched.add(raw)
  layerSet.add(payload.layer)

  if (!sheet.get(payload.currentSelector)) {
    sheet.set(payload.currentSelector, { layer: payload.layer, body: [] })
  }

  (sheet.get(payload.currentSelector)!.body as string[]).push(payload.body)
})

await Promise.all(tokenPromises)
```

这一步很重要，解析token包括解析它的
- [Custom variants](https://github.com/unocss/unocss#custom-variants) 
- [Custom Rules](https://github.com/unocss/unocss#custom-rules) 
- [Shortcuts](https://github.com/unocss/unocss#shortcuts)

解析完之后我们会从parseToken的返回值中拿到
```ts
export interface IParseUtilsResult {
  // 选择器，比如 .text-red
  currentSelector: string
  // 具体内容
  body: string
  // 选择器名字，比如 text-red，前面就没有类选择器了
  selector: string
  // 层级
  layer: string
}
```
关于`layer`的作用，其实就是按照layer的先后顺序来最终渲染css，因为css的先后顺序会影响到样式是否生效，为了解决这个问题，unocss就引入了layer这个概念

可以去文档查看更详细的解释 https://github.com/unocss/unocss#layers

ok 我们继续解析parseToken
```ts
async parseToken(raw: string): Promise<IParseUtilsResult | undefined> {
  if (this.blocked.has(raw)) {
    return
  }

  if (this._cache.has(raw)) {
    return this._cache.get(raw)
  }

  let token = raw

  for (const fn of this.config.preprocess!) {
    token = fn(raw)!
  }

  if (this.isBlocked(token)) {
    this.blocked.add(raw)
    this._cache.set(token, null)
    return
  }

  const { processed, selector } = await this.matchVariants(raw, token)

  if (this.isBlocked(processed!)) {
    this.blocked.add(raw)
    this._cache.set(raw, null)
    return
  }

  const context = {
    rawSelector: raw,
    currentSelector: selector,
    shortcuts: this.config.shortcuts,
    theme: this.config.theme!,
    generator: this,
    ...this.config,
  } as RuleContext<Theme>

  const shortcuts = await this.expandShortcuts(raw, context)

  const util = shortcuts
    ? this.parseShortcutsUtil(processed, shortcuts, context)
    : this.parseUtil(processed, context)

  return util as IParseUtilsResult
}
```

从代码上我们可以看出它是进行如下顺序处理的，如果忘记每个东西是什么了建议再去看一遍文档
- [Utilities Preprocess & Prefixing](https://github.com/unocss/unocss#utilities-preprocess--prefixing)
- [Custom variants](https://github.com/unocss/unocss#custom-variants) 
- [Shortcuts](https://github.com/unocss/unocss#shortcuts)
- [Custom Rules](https://github.com/unocss/unocss#custom-rules) 

首先是**Utilities Preprocess & Prefixing**，这个处理就非常简单了
```ts
let token = raw
for (const fn of this.config.preprocess!)
  token = fn(raw)!
```

就是不断的调用你传进来的函数进行一些处理，最终token会变成你想要的样子

然后是**Custom variants**处理
```ts
async matchVariants(raw: string, token: string): Promise<VariantMatchedResult<Theme>> {
  const variants = new Set<Variant<Theme>>()

  const context: VariantContext<Theme> = {
    rawSelector: raw,
    theme: this.config.theme!,
    generator: this,
  }

  let processed = token
  let selector = raw
  let rule

  for (const v of this.config.variants!) {
    if (variants.has(v)) {
      continue
    }
    const handler = isFunction(v) ? v : v.match
    const result = handler(token, context)

    if (!result || result === token) {
      continue
    }

    processed = isString(result) ? result : result.matcher
    rule = await this.getRule(raw)
    selector = isString(result)
      ? raw
      : (result.selector?.(raw, rule as Rule) as string)

    variants.add(v as Variant<Theme>)
  }

  return { raw, processed, selector }
}
```

其实这些的处理都是非常简单的，调用你配置好的函数，再做一些额外的处理，最终生成你想要的，我分别解释一下返回值的含义，假设我们现在有token`hover:text-red`
- `raw`: 传进来的token
- `processed`: variants中的`matcher`，比如我的token是`hover:text-red`，我的rule是这么定义的
```ts
['text-red', { color: 'red' }]
```
你可以发现如果正常匹配rule的话根本匹配不到，所以真正的规则应该是`hover:`后面的`text-red`，我们把`text-red`叫做`matcher`
```ts
variants: [
  // hover:
  (matcher) => {
    if (!matcher.startsWith('hover:'))
      return matcher
    return {
      // slice `hover:` prefix and passed to the next variants and rules
      matcher: matcher.slice(6),
      selector: s => `${s}:hover`,
    }
  }
],
```
- `selector`: 经过上一层的处理, selector已经从`hover:text-red`变成`hover\:text-red:hover`，最终我们会生成css
```css
.hover\:text-red:hover { color: red }
```


接下来就是处理**Shortcuts**了
```ts
const context = {
  rawSelector: raw,
  currentSelector: selector,
  shortcuts: this.config.shortcuts,
  theme: this.config.theme!,
  generator: this,
} as RuleContext<Theme>

const shortcuts = await this.expandShortcuts(raw, context)
```
**expandShortcuts**
```ts
async expandShortcuts(
  raw: string,
  context: RuleContext<Theme>,
  depth = 5,
): Promise<string[] | undefined> {
  if (depth === 5 && !this.isShortcuts(raw)) {
    return
  }

  if (depth === 0) {
    return []
  }

  for (const shortcut of this.config.shortcuts!) {
    const rule = shortcut[0]
    const matched = isRegExp(rule) ? rule.exec(raw) : raw === rule
    if (!matched) {
      continue
    }
    const shortcuts = await (isFunction(shortcut[1])
      ? shortcut[1](matched as RegExpExecArray, context)
      : shortcut[1])

    if (isString(shortcuts)) {
      const promises = shortcuts
        .split(' ')
        .filter(s => s.length > 0)
        .map((s) => {
          if (this.isShortcuts(s)) {
            return this.expandShortcuts(s, context, depth - 1) || []
          }

          return s
        }) as string[]

      return (await Promise.all(promises)).flat(Infinity)
    }
  }
}
```

解释一下在做什么，比如我有以下的shortcuts配置，为了方便展示，我们就暂且只支持数组形式的表达
```ts
shortcuts: [
  ['red', 'text-red'],
  ['text-main', 'red text-3xl font-bold']
]
```
这个函数的主要作用就是把shortcut展开成rule，如`text-main`会被展开为`text-red text-3xl font-bold`

我们主要看一下`text-main`，会发现它引用了另外一个shortcut: `red`，所以这时候我们就得把引用的shortcuts展开，所以我们用了递归算法，但为了防止一直无限制递归下去，可以设置一个`depth`来限制递归次数最大只能为5
```
const promises = shortcuts
        .split(' ')
        .filter(s => s.length > 0)
        .map((s) => {
          if (this.isShortcuts(s)) {
            return this.expandShortcuts(s, context, depth - 1) || []
          }

          return s
        }) as string[]

return (await Promise.all(promises)).flat(Infinity)
```

最后我们可以拿到一个全是原始值的shortcuts


**解析Rule或Shortcuts**
```ts
const shortcuts = await this.expandShortcuts(raw, context)

const util = shortcuts
  ? this.parseShortcutsUtil(processed, shortcuts, context)
  : this.parseUtil(processed, context)

return util as IParseUtilsResult
```

最后我们可以使用`parseUtil`来拿到对应的rule

```ts
parseUtil(
  raw: string,
  context: Readonly<RuleContext<Theme>>,
): IParseUtilsResult | null {
  const { currentSelector } = context

  const rule = this.getRule(raw)

  if (!rule) {
    return null
  }

  const body = isRegExp(rule[0])
    ? (rule[1] as DynamicMatcher<Theme>)(
        rule[0].exec(raw) as RegExpExecArray,
        context,
      )
    : rule[1]

  if (!body) {
    return null
  }

  return {
    selector: currentSelector,
    layer: rule[2]?.layer || DEFAULT_LAYER,
    currentSelector: generateSelector(currentSelector),
    body: isString(body)
      ? body
      : Object.entries(body)
        .map(([key, val]) => `${key}:${val};`)
        .join(''),
  }
}
```

```ts
parseShortcutsUtil(
  raw: string,
  shortcuts: string[],
  context: Readonly<RuleContext<Theme>>,
): IParseUtilsResult | null {
  const body = shortcuts
    .map(s => this.parseUtil(s, context)?.body)
    .filter(Boolean)
    .join('')

  return {
    selector: context.currentSelector,
    layer: this.getShortcut(raw)?.[2]?.layer || DEFAULT_LAYER,
    currentSelector: generateSelector(context.currentSelector),
    body,
  }
}
```

`parseUtil`主要就是获取rule之后，拿到对应的规则，并把对应的规则转化为字符串，如
```ts
['text-main', { 'color': 'red', 'font-size': '20px' }]
```
会被转化为
```ts
return {
  selector: 'text-main',
  layer: DEFAULT_LAYER,
  currentSelector: '.text-main',
  body: 'color:red;font-size:20px;'
}
```
`parseShortcutsUtil`其实就是做一层遍历

最后我们返回出去，回到`parseToken`中

```ts
const payload = await this.parseToken(raw)

if (!payload)
  return

matched.add(raw)
layerSet.add(payload.layer)

if (!sheet.get(payload.currentSelector)) {
  sheet.set(payload.currentSelector, { layer: payload.layer, body: [] })
}

(sheet.get(payload.currentSelector)!.body as string[]).push(payload.body)
```

此时我们的token就算是解析完成了，接下来就是生成css了，在此之前，我们需要组合一下各层的layer

```ts
const layers = Array.from(new Set(this.config
  .rules!.map(r => r[2]?.layer)
  .filter(Boolean)
  .concat(Array.from(layerSet)))) as string[]
```

按道理来说这里应该有一个排序算法的，unocss是按照字母的顺序来排序layer，但我觉得用用户配置layer的顺序来默认排序是比较合适的，这里就仁者见仁智者见智了

最后我们再通过layer生成css

```ts
const layers = Array.from(new Set(this.config
  .rules!.map(r => r[2]?.layer)
  .filter(Boolean)
  .concat(Array.from(layerSet)))) as string[]

const layerCache: Record<string, string> = {}

const getLayer = (layer: string) => {
  if (layerCache[layer])
    return layerCache[layer]

  const css = Array.from(sheet)
    .filter(
      ([, { layer: cssLayer }]) => cssLayer === (layer || DEFAULT_LAYER),
    )
    .map(([selector, { body }]) => {
      return `${selector.replaceAll(':', '\:')}{${isString(body) ? body : body.join('')}}`
    })
    .concat(layerPreflights.get(layer) ?? [])
    .join('\n')

  const layerMark = css ? `/** layer ${layer} **/\n${css}` : ''
  return (layerCache[layer] = layerMark)
}

const getLayers = (includes = layers, excludes?: string[]) => {
  return includes
    .filter(i => !excludes?.includes(i))
    .map(i => getLayer(i) || '')
    .join('\n')
    .trim()
}

return {
  get css() {
    return getLayers()
  },
  getLayer,
  getLayers,
}
```

我们可以跑个单测测一下
```ts
test('layer', async() => {
  const uno = createGenerator({
    rules: [
      ['a', { name: 'bar1', age: '18' }, { layer: 'default' }],
      ['b', { name: 'bar2' }, { layer: 'b' }],
      [/^c(\d+)$/, ([, d]) => ({ name: d }), { layer: 'c' }],
      [/^d(\d+)$/, ([, d]) => `/* RAW ${d} */`, { layer: 'd' }],
    ],
    shortcuts: [
      ['abcd', 'a b c d', { layer: 'abcd' }],
      ['ab', 'a b'],
    ],
  })

  const { css } = await uno.generate('a b c d c1 d2 abcd ab')

  expect(css).toMatchInlineSnapshot(`
    "/** layer default **/
    .a{name:bar1;age:18;}
    .ab{name:bar1;age:18;name:bar2;}
    /** layer b **/
    .b{name:bar2;}
    /** layer c **/
    .c1{name:1;}
    /** layer d **/
    .d2{/* RAW 2 */}
    /** layer abcd **/
    .abcd{name:bar1;age:18;name:bar2;}"
  `)
})
```

可以发现`toMatchInlineSnapshot`生成的css是非常的完美的

附上单测跑过的图

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/059942204b8e4b8f9c881b1b2f3de9d9~tplv-k3u1fbpfcp-watermark.image?)


OK, `@unocss/core`的主要原理就讲解完毕了，可以发现核心思想也并不是很难，只是实现各种功能有些麻烦

关于具体的`mini-unocss`，大家可以去我仓库中查看哦，https://github.com/SnowingFox/mini-unocss/tree/master/packages/core
