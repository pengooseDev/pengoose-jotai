# @pengoose/jotai

<div align="center">
  <div>
    <a href="https://www.npmjs.com/package/@pengoose/jotai">
      <img src="https://img.shields.io/npm/v/@pengoose/jotai?style=for-the-badge" alt="npm version" />
    </a>
  </div>

<h3>Reactの状態をJotaiを使って簡単かつ強力に管理するConvention manager</h3>
  <picture width="350">
    <source media="(prefers-color-scheme: light)" srcset="https://i.imgur.com/lPN6qxb.png" width="550">
    <source media="(prefers-color-scheme: dark)" srcset="https://i.imgur.com/N5aPzdJ.png" width="550">
    <img alt="IMAGE" src="https://i.imgur.com/lPN6qxb.png">
  </picture>
  <h4> * Illustration created by <a href="https://twitter.com/takuan1469414">沢庵</a> </h4>

[English](../README.md) | [한국어](./README_KO.md) | [日本語](./README_JA.md)

</div>

## 概要

`@pengoose/jotai`は Jotai を使用して React アプリケーションの状態を管理する`Convention manager`です。アプリケーションの状態を定義し、管理するためのシンプルで構造化された方法を提供し、状態ロジックを整理し、メンテナンスしやすくします。一連の規則に従うことで、理解しやすく、メンテナンスしやすい一貫した拡張可能な状態管理システムを作成できます。

## インストール

npm を使用する場合:

```sh
npm install @pengoose/jotai
```

yarn を使用する場合:

```sh
yarn add @pengoose/jotai
```

## 使い方 (ステップバイステップ)

1. atom で管理する状態のインターフェースを定義します。
2. `AtomManager`のジェネリック型にインジェクトします。
3. AtomManager(抽象クラス)を継承して状態を管理するクラスを作成します。

### Step1: atom で管理する状態のインターフェースを定義

```ts
// example/types.ts
export interface Music {
  id: string;
  title: string;
  thumbnail: string;
  url: string;
}

export interface PlaylistStatus {
  playlist: Music[];
  index: number;
}
```

### Step2: atomManager クラスを作成

- `AtomManager`クラスを拡張して状態マネージャを作成します。
- `AtomManager`クラスで selectors(Getter)と actions(Setter)を実装する必要があります。

```ts
import { atom } from 'jotai';
import { AtomManager } from '@pengoose/jotai';
import { PlaylistStatus, Music } from '@/types';

// 1. AtomManagerを拡張して状態マネージャを作成します。
export class Playlist extends AtomManager<PlaylistStatus> {
  // 2. selectorsを実装します。
  public selectors = {
    playlist: atom((get) => {
      const { playlist } = get(this.atom);

      return playlist;
    }),

    currentMusic: atom((get) => {
      const { playlist, index } = get(this.atom);

      return playlist[index];
    }),

    // ... other selectors
  };

  // 3. actionsを実装します。
  public actions = {
    add: atom(null, (get, set, music: Music) => {
      const { playlist } = get(this.atom);

      if (playlist.some(({ id }) => id === music.id)) return;

      set(this.atom, (prev: PlaylistStatus) => ({
        ...prev,
        playlist: [...prev.playlist, music],
      }));
    }),

    // ... other actions
  };

  // 4. 必要に応じてオブジェクト内で使用する追加のメソッドを実装します。
  private isEmpty(playlist: Music[]) {
    return playlist.length === 0;
  }

  private isFirstMusic(index: number) {
    return index === 0;
  }
}

// 5. Playlistクラスのインスタンスを作成します。
const initialState: PlaylistStatus = {
  playlist: [],
  index: 0,
};

export const playlistManager = new Playlist(initialState);
```

---

### Step3: `useManager`フックで AtomManager インスタンスをラップする

- `useManager`フックは AtomManager インスタンス内部の抽象化された selectors と actions をそれぞれ useAtomValue と useSetAtom フックに変換し、それに対する型を推論してユーザーに返します。

```tsx
// usePlaylist.ts
import { useManager } from '@pengoose/jotai';
import { playlistManager } from '@/viewModel';

export const usePlaylist = () => {
  const {
    selectors: { playlist, currentMusic },
    actions: { play, next, prev, add, remove },
  } = useManager(playlistManager); // 😀👍

  return {
    // Getters(Selectors)
    playlist,
    currentMusic,

    // Setters(Actions)
    play,
    next,
    prev,
    add,
    remove,
  };
};
```

### ⛳️ `useManager`フックを使用しない場合 🥲

- `useManager`を使用せずに Jotai の useAtomValue と useSetAtom フックを使用して AtomManager インスタンスの状態を取得および設定できます。ただし、少し手間がかかります。 😨

```tsx
// usePlaylist.ts
import { useAtomValue, useSetAtom } from 'jotai';
import { playlistManager } from '@/viewModel';

export const usePlaylist = () => {
  const {
    selectors: { playlist, currentMusic },
    actions: { play, next, prev, add, remove },
  } = playlistManager;

  return {
    // Getters(Selectors) 😨
    playlist: useAtomValue(playlist),
    currentMusic: useAtomValue(currentMusic),

    // Setters(Actions) 😨
    play: useSetAtom(play),
    next: useSetAtom(next),
    prev: useSetAtom(prev),
    add: useSetAtom(add),
    remove: useSetAtom(remove),
  };
};
```

---

### Step4: コンポーネントでカスタムフックを使用する 🚀

```tsx
// Playlist.tsx
import { usePlaylist } from '@/hooks';
import { Music } from '@/types';

export const Playlist = () => {
  const { playlist, currentMusic, play, next, prev, add, remove } =
    usePlaylist();

  return (
    <div>
      <h1>Playlist</h1>
      <ul>
        {playlist?.map((music) => {
          const { id, title, thumbnail } = music;

          return (
            <li key={id}>
              <img src={thumbnail} alt={title} />
              <p>{title}</p>
              <button onClick={() => remove(music)}>Remove</button>
            </li>
          );
        })}
      </ul>
      <button onClick={() => add(currentMusic)}>Add to Playlist</button>
      <button onClick={() => play(currentMusic)}>Play</button>
      <button onClick={prev}>Prev</button>
      <button onClick={next}>Next</button>
    </div>
  );
};
```

---

## 要約

`AtomManager`クラスはカスタムフックと組み合わせて状態管理ロジックをカプセル化し、コンポーネントで簡単に使用できるように設計されています。

> フロー: Class(AtomManager) --> custom hook --> Component(View)

<div align="center">
  😗👍
</div>

## Contributing

貢献はいつでも歓迎します！主要な変更の場合は、まず issue を作成して議論していただけると幸いです。;)
