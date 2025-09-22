<template>
  <div class="markdown-renderer">
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
import { ref, shallowRef, onMounted } from 'vue'
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
    }
  },
  setup(props) {
    // 使用 shallowRef 来避免不必要的深层响应式处理
    const markdownRenderer = ref(null)
    
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
    onMounted(() => {
      // 组件已挂载
    })
    
    return {
      markdownRenderer,
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
}

.rendered-content {
  min-width: auto;
  width: 100%;
  display: block;
  word-break: normal;
  overflow-wrap: break-word;
  hyphens: none;
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
}

.markdown-renderer h1 { font-size: 2em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.markdown-renderer h2 { font-size: 1.5em; border-bottom: 1px solid #eee; padding-bottom: 0.3em; }
.markdown-renderer h3 { font-size: 1.3em; }
.markdown-renderer h4 { font-size: 1.1em; }
.markdown-renderer h5 { font-size: 1em; }
.markdown-renderer h6 { font-size: 0.9em; color: #777; }

/* 引用块样式 */
.markdown-renderer blockquote {
  padding: 0 1em;
  color: #57606a;
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
}

/* 列表样式 */
.markdown-renderer ul,
.markdown-renderer ol {
  text-align: left;
  padding-left: 2em;
  margin-top: 0;
  margin-bottom: 1em;
}

.markdown-renderer ul li,
.markdown-renderer ol li {
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
}

.markdown-renderer table th {
  font-weight: 600;
  background-color: #f6f8fa;
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
.markdown-renderer pre {
  background-color: #f6f8fa;
  border-radius: 6px;
  padding: 16px;
  overflow: auto;
  margin: 1em 0;
  line-height: 1.45;
  font-size: 85%;
}

.markdown-renderer code {
  background-color: rgba(27,31,35,0.05);
  border-radius: 3px;
  padding: 0.2em 0.4em;
  font-size: 85%;
  font-family: ui-monospace, SFMono-Regular, "SF Mono", Consolas, "Liberation Mono", Menlo, monospace;
}

.markdown-renderer pre code {
  background-color: transparent;
  padding: 0;
  font-size: 100%;
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
</style>