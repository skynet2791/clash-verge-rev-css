# Clash Verge Rev 国旗图标渲染与 CSS 定制注意事项

在使用自定义 CSS 美化 Clash Verge Rev 客户端（特别是基于 Shadcn 风格进行深度定制）时，国旗和代理节点图标的渲染极易受全局样式干扰。经过多次排查，现将踩坑记录与最佳实践总结如下：

## 1. 警惕全局字体覆盖（Ligatures 失效问题）
**现象**：节点前的彩色国旗（如 🇨🇳、🇺🇸）变成了纯文本的代号缩写（如 `cn`、`us`、`HK`）。
**原因**：Clash Verge Rev 通常利用特殊的 Emoji 字体或内建图标字体（Icon Fonts），通过**连字（Ligatures）**特性将字母代号渲染为彩色国旗。
**错误做法**：在全局 CSS 中使用 `* { font-family: 'Inter', ... !important; }`。这会强制覆盖节点原本绑定的图标字体，导致浏览器渲染回纯文本。
**最佳实践**：
- 仅对 `body` 施加常规 `font-family`。
- 对于 `div, span, a, button` 等元素使用 `font-family: inherit`。
- 坚决避免使用带 `!important` 的通配符 `*` 字体覆盖。

## 2. 避免宽泛的伪元素清除（::before 误伤）
**现象**：国旗图标完全消失。
**原因**：在定制高质感卡片时，为了移除原生主题（例如选中状态左侧自带的蓝色指示竖条），很多主题会大范围隐藏伪元素。如果使用类似 `.active::before { display: none !important; }` 的宽泛规则，极有可能连带删除了用 `::before` 渲染背景图或图标的国旗元素。
**最佳实践**：
使用精准对齐的组件封装选择器（如 `.MuiListItemButton-root.Mui-selected::before`），绝不滥用通用的 `.active` 或 `.selected` 基础类名。

## 3. 国旗图标的“单色化”极简改造方案
**诉求**：让所有国旗默认呈现清爽的灰色单色（Monochrome），仅在交互时（Hover/Active）恢复色彩，以降低画面色彩杂乱感。
**挑战**：在某些配置下，Clash Verge 不会给图标添加独立的外层 `<span>`，而是直接将 `🇨🇳` Emoji 字符与节点名称混排在同一个纯文本 `Typography` 标签中。
**巧妙解法**：
对包揽文本的**整层容器**施加灰度滤镜。由于主题的普通文本本身就是黑/深灰色，`grayscale(100%)` 对其毫无影响，却能精准地把文字内部的彩色字符（Emoji国旗）抽干颜色。
```css
/* 对包含文字和 Emoji 的通用容器施加灰度滤镜 */
.layout-content__right [class*="card"] .MuiTypography-root,
.layout-content__right [class*="card"] .MuiListItemText-root {
  filter: grayscale(100%) opacity(0.85) !important;
  transition: filter 0.25s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.25s cubic-bezier(0.4, 0, 0.2, 1) !important;
}

/* 在悬停或选中状态时恢复彩色 */
.layout-content__right [class*="card"]:hover .MuiTypography-root,
.layout-content__right [class*="card"].active .MuiTypography-root {
  filter: grayscale(0%) opacity(1) !important;
}
```
