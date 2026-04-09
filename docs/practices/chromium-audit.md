# Chromium 安全审计流程

**日期**：2026-04-09  
**关联**：[[工兵陷阱]]  
**报告**：`/workspace/chromium/BOOKMARK_AUDIT_REPORT.md`

---

## 发现的漏洞（汇总）

所有问题都是同一个根因：**递归反序列化无深度限制**

| 路径 | 函数 | 平台 | 攻击面 |
|------|------|------|--------|
| bookmark_node_data.cc | ReadFromLegacyPickle / FromPickle | Win/Linux | 剪贴板粘贴 |
| bookmark_codec.cc | DecodeNode ↔ DecodeChildren | 全平台 | Bookmarks JSON 文件 |
| bookmark_pasteboard_helper_mac.mm | ConvertNSArrayToElements | macOS | 剪贴板粘贴 |
| managed_bookmarks_policy_handler.cc | FilterBookmarks | 全平台 | Enterprise 策略 |

## 修复方案

统一加 `kMaxBookmarkNestingDepth = 200` 深度检查。

Patches：
- `fix_bookmark_pickle_depth.patch`
- `fix_bookmark_codec_depth.patch`
