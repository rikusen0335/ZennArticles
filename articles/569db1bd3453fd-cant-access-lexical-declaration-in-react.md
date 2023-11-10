---
title: "ReactでCan't access lexical declarationが出た時"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react"]
published: true
---

### React で`ReferenceError: can't access lexical declaration 'Child' before initialization`が出た時

Child はコンポーネントだとします。

components/organisms/index.ts

```
export * from './Parent'
export * from './Child'
```

Parent.tsx

```
import { Child } from '@/components/organisms'

export const Parent = () => {
  return <Child />
}
```

Child.tsx

```
export const Child = () => {
  return <p>hoge</p>
}
```

これだとエラーになります。
この場合、Parent の import 文を直してあげることで修正できます

Parent.tsx

```
import { Child } from '@/components'

export const Parent = () => {
  return <Child />
}
```

`organisms`を消したら直ります。消さなくても正常に動くと思っていたので、なぜかはわかってないですがいったんメモで
