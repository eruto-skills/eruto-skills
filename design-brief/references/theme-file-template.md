# Theme ファイル テンプレート

## 出力先

```
tech/remotion/src/themes/<kebab-name>.ts
```

export は `tech/remotion/src/index.ts` に追加する。

## テンプレート

```typescript
/**
 * <theme-name>: <一行説明>
 *
 * フォント: <フォント名>（<提供元>）
 * Google Fonts URL（Root.tsx に追加が必要な場合）:
 *   https://fonts.googleapis.com/css2?family=...
 */
import type { Theme } from '../theme';

export const <camelCaseName>: Theme = {
  id: '<kebab-case-name>',

  bg:       '<hex>',
  fg:       '<hex>',
  fgMuted:  '<hex>',
  accent:   '<hex>',
  surface:  '<hex>',
  border:   '<hex>',

  font: {
    sans: '"<Font Name>", sans-serif',
    mono: '"<Mono Font>", monospace',
    // display: '"<Display Font>", serif',  // 見出しフォントが異なる場合
  },

  fontSize: {
    xs: 22, sm: 30, base: 36, lg: 46,
    xl: 56, '2xl': 72, '3xl': 100, hero: 140,
  },

  radius: { sm: <n>, md: <n>, lg: <n>, full: 9999 },
  motion: { fast: <n>, normal: <n>, slow: <n> },

  effects: {
    // ダークテーマのグロー（glow がある場合）:
    // glow: (color: string, intensity = 1) =>
    //   `0 0 ${20 * intensity}px ${color}40, 0 0 ${60 * intensity}px ${color}20`,
    //
    // ライトテーマのシャドウ:
    // shadow: '0 4px 24px rgba(0,0,0,0.10)',
  },
};
```

## 命名規則

| 項目 | 形式 | 例 |
|------|------|----|
| ファイル名 | `<kebab-name>.ts` | `cs-crypto.ts` |
| export 名 | `camelCase` | `csCrypto` |
| `id` | `kebab-case`（ファイル名と一致） | `cs-crypto` |

## index.ts への追加

```typescript
// tech/remotion/src/index.ts に追加する行:
export { <camelCaseName> } from './themes/<kebab-name>';
```

コメントに番組名・用途を一言添えると他のテーマと見分けやすい:
```typescript
export { csCrypto } from './themes/cs-crypto';     // ゆるCS 暗号補講
```

## スケール値の目安

### radius（角丸）
| スタイル | sm | md | lg |
|----------|----|----|-----|
| かわいい・丸め | 12 | 22 | 36 |
| 標準 | 6 | 12 | 20 |
| 落ち着き・フォーマル | 2 | 6 | 12 |
| シャープ（ゲーム系） | 0 | 4 | 8 |

### motion（フレーム数 @ 30fps）
| スタイル | fast | normal | slow |
|----------|------|--------|------|
| テンポよく（ゲームUI） | 8 | 14 | 22 |
| 標準 | 12 | 20 | 36 |
| ゆったり・余韻あり | 15 | 28 | 52 |

### fontSize（基本は変えない）
通常は `fontSize` をデフォルトのまま使う。
フォントによって視覚的な大きさが変わる場合のみ `base` を ±4 程度調整する。

## よくあるパターン

### ダークテーマ（グロー付き）
```typescript
bg: '#08091A', fg: '#E8EEF8', fgMuted: '#7090B0',
accent: '#F5C840', surface: '#0F1428', border: '#243560',
effects: {
  glow: (c, i = 1) => `0 0 ${20*i}px ${c}50, 0 0 ${60*i}px ${c}25`,
},
```

### ライトテーマ（学術・クリーン）
```typescript
bg: '#FFFFFF', fg: '#1A1A2E', fgMuted: '#6B7280',
accent: '#1A78C2', surface: '#F0F8FF', border: '#CBD5E1',
effects: {
  shadow: '0 2px 16px rgba(0,0,0,0.08)',
},
```

### ダーク+シャープ（真剣なCS・テクニカル）
```typescript
bg: '#0D1117', fg: '#E6EDF3', fgMuted: '#8B949E',
accent: '#58A6FF', surface: '#161B22', border: '#30363D',
// radius: sm:0, md:4, lg:8 でシャープに
// motion: fast:10, normal:18, slow:30 でテンポよく
```
