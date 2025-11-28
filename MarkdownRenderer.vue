<template>
  <div 
    class="markdown-renderer" 
    :class="{ 'streaming-mode': streaming }"
    ref="rootEl"
  >
    <VueMarkdown 
      ref="markdownRenderer"
      :content="content"
      :plugin="plugins"
      :fence-plugin="fencePlugins"
      class="rendered-content"
    />
  </div>
</template>

<script>
import { ref, onMounted, watch, nextTick } from 'vue'
import VueMarkdown from 'vue-markdown-stream'
import 'vue-markdown-stream/dist/index.css'
import { full as emoji } from 'markdown-it-emoji'
import markdownItTexmath from 'markdown-it-texmath'
import 'katex/dist/katex.min.css'
// 如果有 MermaidChart.vue 文件，可以导入它
import MermaidChart from './MermaidChart.vue'
// 导入 Echarts 组件
import Echarts from './Echarts.vue'

export default {
  name: 'MarkdownRenderer',
  components: {
    VueMarkdown
  },
  props: {
    content: {
      type: String,
      required: true
    },
    streaming: {
      type: Boolean,
      default: false
    }
  },
  setup(props) {
    const markdownRenderer = ref(null)
    const rootEl = ref(null)
    
    // 启用 emoji 与 LaTeX(KaTeX) 渲染（行内与块级）
    // markdown-it-katex 默认支持 $...$ 与 $$...$$ 语法
    const plugins = [
      [emoji],
      [markdownItTexmath, {
        engine: 'katex',
        delimiters: 'dollars',
        katexOptions: {
          throwOnError: false,
          strict: false,
          output: 'htmlAndMathml'
        }
      }]
    ]
    
    // 定义 fence 插件
    const fencePlugins = {
      // 添加 Mermaid 支持（兼容两种 fence 名称）
      mermaid: MermaidChart,
      mermaidChart: MermaidChart,
      // 添加 Echarts 支持
      echarts: Echarts
    }
    
    // 如果需要在组件挂载后执行某些操作
    const enhanceCodeBlocks = () => {
      nextTick(() => {
        const container = rootEl.value
        if (!container) return

        const codeBlocks = container.querySelectorAll('pre')
        codeBlocks.forEach((pre) => {
          if (pre.dataset.enhanced === 'true') return
          const codeElement = pre.querySelector('code')

          const wrapper = document.createElement('div')
          wrapper.className = 'code-block-container'

          const header = document.createElement('div')
          header.className = 'code-block-header'

          const languageLabel = document.createElement('span')
          languageLabel.className = 'code-block-language'
          const languageMatch = codeElement?.className?.match(/language-([\w+-]+)/i)
          languageLabel.textContent = languageMatch ? languageMatch[1].toUpperCase() : 'CODE'

          const copyButton = document.createElement('button')
          copyButton.type = 'button'
          copyButton.className = 'code-block-copy-button'
          copyButton.textContent = '复制'

          const resetButtonState = (text = '复制') => {
            copyButton.textContent = text
            copyButton.classList.remove('copied', 'error')
          }

          copyButton.addEventListener('click', async () => {
            const codeText = codeElement?.innerText ?? ''
            if (!codeText) return

            try {
              await navigator.clipboard.writeText(codeText)
              copyButton.textContent = '已复制'
              copyButton.classList.add('copied')
            } catch (error) {
              copyButton.textContent = '复制失败'
              copyButton.classList.add('error')
              console.error('复制代码失败:', error)
            }

            setTimeout(() => resetButtonState(), 2000)
          })

          header.appendChild(languageLabel)
          header.appendChild(copyButton)

          pre.dataset.enhanced = 'true'
          pre.classList.add('code-block')

          pre.parentNode?.insertBefore(wrapper, pre)
          wrapper.appendChild(header)
          wrapper.appendChild(pre)
        })

        const links = container.querySelectorAll('a[href]')
        links.forEach((anchor) => {
          if (anchor.dataset.enhanced === 'true') return

          // 检查链接是否包含图片标签,如果是图片链接则跳过处理
          const hasImage = anchor.querySelector('img')
          if (hasImage) {
            // 图片链接不做处理,让图片正常显示
            anchor.dataset.enhanced = 'true'
            return
          }

          anchor.setAttribute('target', '_blank')
          anchor.setAttribute('rel', 'noopener noreferrer')

          const handleClick = (event) => {
            const href = anchor.getAttribute('href')
            if (!href) return
            event.preventDefault()
            window.open(href, '_blank', 'noopener')
          }

          anchor.addEventListener('click', handleClick)
          anchor.dataset.enhanced = 'true'
        })
      })
    }

    onMounted(() => {
      enhanceCodeBlocks()
    })

    watch(
      () => props.content,
      () => {
        enhanceCodeBlocks()
      }
    )
    
    return {
      markdownRenderer,
      rootEl,
      plugins,
      fencePlugins
    }
  }
}
</script>

<style>
/* 保留原有的样式 */
.markdown-renderer {
  word-break: normal;
  overflow-wrap: break-word;
  hyphens: none;
  line-height: 1.6;
  text-align: left;
  overflow-x: auto; /* 横向可滚动，容纳宽图表 */
  color: #1a202c; /* 默认深色文字 */
}

.rendered-content {
  min-width: auto;
  width: 100%;
  display: block;
  word-break: normal;
  overflow-wrap: break-word;
  hyphens: none;
  color: #1a202c; /* 深色文字 */
}

/* 让图表块可以根据内容自适应宽度，同时不超出容器（横向滚动） */
.echarts-block,
.mermaid-block {
  display: inline-block;
  max-width: 100%;
}

/* 标题样式 */
.markdown-renderer h1,
.markdown-renderer h2,
.markdown-renderer h3,
.markdown-renderer h4,
.markdown-renderer h5,
.markdown-renderer h6 {
  margin-top: 1.5em;
  margin-bottom: 0.5em;
  font-weight: 600;
  line-height: 1.25;
  text-align: left;
  color: #1a202c; /* 深色文字 */
}

.markdown-renderer h1 { font-size: 2em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.markdown-renderer h2 { font-size: 1.5em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.markdown-renderer h3 { font-size: 1.3em; }
.markdown-renderer h4 { font-size: 1.1em; }
.markdown-renderer h5 { font-size: 1em; }
.markdown-renderer h6 { font-size: 0.9em; color: #4a5568; } /* h6 使用稍浅的深灰色 */

/* 引用块样式 */
.markdown-renderer blockquote {
  padding: 0 1em;
  color: #4a5568; /* 引用块使用稍浅的深灰色 */
  border-left: 0.25em solid #d0d7de;
  margin: 0 0 16px 0;
}

.markdown-renderer blockquote p {
  margin-top: 0;
  margin-bottom: 0;
}

/* 段落样式 */
.markdown-renderer p {
  margin-top: 0;
  margin-bottom: 1em;
  text-align: left;
  word-break: normal;
  overflow-wrap: break-word;
  hyphens: none;
  line-height: 1.6;
  color: #1a202c; /* 深色文字 */
}

/* 列表样式 */
.markdown-renderer ul,
.markdown-renderer ol {
  text-align: left;
  padding-left: 2em;
  margin-top: 0;
  margin-bottom: 1em;
  color: #1a202c; /* 确保列表容器也有颜色 */
}

.markdown-renderer ul li,
.markdown-renderer ol li {
  margin-bottom: 0.25em;
  color: #1a202c; /* 🔥 修复：明确设置列表项文字颜色为深色 */
  line-height: 1.6;
}

/* 确保列表项的标记（圆点/数字）也是深色 */
.markdown-renderer ul li::marker,
.markdown-renderer ol li::marker {
  color: #1a202c;
}

/* 嵌套列表样式 */
.markdown-renderer ul ul,
.markdown-renderer ul ol,
.markdown-renderer ol ul,
.markdown-renderer ol ol {
  margin-top: 0.25em;
  margin-bottom: 0.25em;
}

/* 表格样式 */
.markdown-renderer table {
  border-collapse: collapse;
  margin: 1em 0;
  display: block;
  width: max-content;
  max-width: 100%;
  overflow: auto;
  text-align: left;
  border-spacing: 0;
  border: 1px solid #dfe2e5;
  border-radius: 6px;
}

.markdown-renderer table th,
.markdown-renderer table td {
  border: 1px solid #dfe2e5;
  padding: 6px 13px;
  color: #1a202c;
}

.markdown-renderer table th {
  font-weight: 600;
  background-color: #f6f8fa;
  color: #1a202c;
}

.markdown-renderer table tr {
  background-color: #fff;
  border-top: 1px solid #c6cbd1;
}

.markdown-renderer table tr:nth-child(2n) {
  background-color: #f6f8fa;
}

/* 图片样式 */
.markdown-renderer img {
  max-width: 100%;
  box-sizing: content-box;
  background-color: #fff;
  border-radius: 6px;
  box-shadow: 0 1px 2px rgba(0,0,0,0.1);
}

/* 分隔线样式 */
.markdown-renderer hr {
  height: 0.25em;
  padding: 0;
  margin: 24px 0;
  background-color: #e1e4e8;
  border: 0;
  border-radius: 0.125em;
}

/* 代码块样式增强 */

.code-block-container {
  position: relative;
  margin: 1.5em 0;
  border-radius: 12px;
  background: linear-gradient(180deg, #fafbff 0%, #f4f7fb 100%);
  border: 1px solid rgba(148, 163, 184, 0.28);
  box-shadow: 0 14px 30px -20px rgba(15, 23, 42, 0.35);
  overflow: hidden;
}

.code-block-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.65em 1.05em;
  background: rgba(255, 255, 255, 0.9);
  border-bottom: 1px solid rgba(148, 163, 184, 0.22);
  font-size: 0.75rem;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #4a5568;
}

.code-block-language {
  font-weight: 600;
  color: #2d3748;
  background: rgba(74, 144, 226, 0.14);
  border: 1px solid rgba(74, 144, 226, 0.3);
  border-radius: 8px;
  padding: 0.2em 0.7em;
  letter-spacing: 0.04em;
}

.code-block-copy-button {
  border: 1px solid rgba(148, 163, 184, 0.38);
  background: rgba(255, 255, 255, 0.85);
  color: #4a5568;
  border-radius: 8px;
  padding: 0.3em 0.9em;
  font-size: 0.75rem;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  gap: 0.35em;
  box-shadow: 0 4px 12px -10px rgba(74, 144, 226, 0.8);
}

.code-block-copy-button:hover {
  background: rgba(123, 182, 246, 0.18);
  border-color: rgba(74, 144, 226, 0.4);
  transform: translateY(-1px);
}

.code-block-copy-button:active {
  transform: translateY(0);
}

.code-block-copy-button.copied {
  background: rgba(72, 187, 120, 0.18);
  color: #166534;
  border-color: rgba(72, 187, 120, 0.35);
}

.code-block-copy-button.error {
  background: rgba(245, 101, 101, 0.18);
  color: #9b1c1c;
  border-color: rgba(245, 101, 101, 0.35);
}

.code-block {
  background-color: transparent;
  color: inherit;
  border-radius: 0;
  padding: 1.15em 1.35em;
  overflow: auto;
  margin: 0;
  line-height: 1.65;
  font-size: 0.92rem;
  border-top: 1px solid rgba(148, 163, 184, 0.16);
  font-family: 'Fira Code', 'JetBrains Mono', 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;
}

.markdown-renderer code {
  background-color: rgba(15, 23, 42, 0.08);
  border-radius: 6px;
  padding: 0.2em 0.45em;
  font-size: 0.9rem;
  font-family: 'Fira Code', 'JetBrains Mono', 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;
  color: #0f172a;
}

.markdown-renderer pre code {
  background-color: transparent;
  padding: 0;
  font-size: 1rem;
}

/* KaTeX 渲染：为块级公式添加垂直间距，并优化滚动 */
.markdown-renderer .katex-display {
  display: block; /* 确保作为块级元素，以便应用垂直外边距 */
  text-align: left;
  overflow-x: auto;
  overflow-y: auto;
  width: 100%; /* 确保宽度为 100% */;
  -webkit-overflow-scrolling: touch;
  padding: 0.5em 0.25em; /* 增加内边距 */
  margin-top: 1em;    /* 添加上外边距，解决与上方元素的重叠 */
  margin-bottom: 1em; /* 添加下外边距，解决与下方元素的重叠 */
  white-space: normal;
}

/* 流式输出期间隐藏图片，避免不完整URL导致闪烁 */
.markdown-renderer.streaming-mode img {
  display: none;
}
</style>
