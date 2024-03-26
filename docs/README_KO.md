# @pengoose/jotai

<div align="center">

<h3>쉽고 강력한 Jotai 상태관리 Convention manager</h3>
  <div>
    <a href="https://www.npmjs.com/package/@pengoose/jotai">
      <img src="https://img.shields.io/npm/v/@pengoose/jotai?style=for-the-badge" alt="npm version" />
    </a>
  </div>

  <picture width="350">
    <source media="(prefers-color-scheme: light)" srcset="https://i.imgur.com/lPN6qxb.png" width="550">
    <source media="(prefers-color-scheme: dark)" srcset="https://i.imgur.com/N5aPzdJ.png" width="550">
    <img alt="IMAGE" src="https://i.imgur.com/lPN6qxb.png">
  </picture>
  <h4> * Illustration created by <a href="https://twitter.com/takuan1469414">沢庵</a></h4>

[English](../README.md) | [한국어](./README_KO.md) | [日本語](./README_JA.md)

</div>

## 개요

`@pengoose/jotai`는 Jotai를 사용하여 React 애플리케이션의 상태를 관리하는 `Convention manager`입니다. 응용 프로그램의 상태를 정의하고 관리하는 간단하고 구조화된 방법을 제공하여 상태 로직을 조직화하고 유지 관리하기 쉽게 만듭니다. 일련의 규칙을 따르면 이해하고 유지 관리하기 쉬운 일관된 확장 가능한 상태 관리 시스템을 만들 수 있습니다.

## 설치

npm을 사용하는 경우:

```sh
npm install @pengoose/jotai
```

yarn을 사용하는 경우:

```sh
yarn add @pengoose/jotai
```

## 적용법 (단계별)

1. atom으로 관리할 상태의 인터페이스를 선언합니다.
2. `AtomManager`의 제네릭 타입에 주입합니다.
3. 하나의 Class에 AtomManager(추상 클래스)를 상속받아 상태를 관리하는 클래스를 만듭니다.

### Step1: Atom으로 관리할 상태 인터페이스 정의

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

### Step2: atomManager Class 만들기

- `AtomManager` 클래스를 상속받아 상태 관리자를 만듭니다.
- `AtomManager` 클래스에서 selectors(Getter)와 actions(Setter)를 구현해야 합니다.

```ts
import { atom } from 'jotai';
import { AtomManager } from '@pengoose/jotai';
import { PlaylistStatus, Music } from '@/types';

// 1. AtomManager를 확장하여 상태 관리자를 만듭니다.
export class Playlist extends AtomManager<PlaylistStatus> {
  // 2. selectors를 구현합니다.
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

  // 3. actions를 구현합니다.
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

  // 4. 원하는 경우 객체 내부에서 사용할 추가적인 메소드를 구현합니다.
  private isEmpty(playlist: Music[]) {
    return playlist.length === 0;
  }

  private isFirstMusic(index: number) {
    return index === 0;
  }
}

// 5. Playlist 클래스의 인스턴스를 생성합니다.
const initialState: PlaylistStatus = {
  playlist: [],
  index: 0,
};

export const playlistManager = new Playlist(initialState);
```

---

### Step3: `useManager` 훅으로 AtomManager 인스턴스를 래핑하기

- `useManager` 훅은 AtomManager 인스턴스 내부에 추상화 된 selectos와 actions를 각각 useAtomValue와 useSetAtom 훅으로 변환하고, 이에 대한 타입을 추론하여 사용자에게 반환합니다.

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

### ⛳️ `useManager` 훅을 사용하지 않는 경우 🥲

- useManager 없이 Jotai의 useAtomValue와 useSetAtom 훅을 사용하여 AtomManager 인스턴스의 상태를 가져오고 설정할 수 있습니다. 하지만, 사용하기에는 조금 번거롭습니다. 😨

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

### Step4: 컴포넌트에서 커스텀 훅 사용하기 🚀

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

## 요약

`AtomManager` 클래스는 커스텀 훅과 함께 사용하여 상태 관리 로직을 캡슐화하고 컴포넌트에서 쉽게 사용할 수 있도록 설계되었습니다.

> Flow: Class(AtomManager) --> custom hook --> Component(View)

<div align="center">
  😗👍
</div>

## Contributing

기여는 언제나 환영입니다! 주요 변경 사항의 경우 먼저 issue를 생성하여 토의해주시면 감사하겠습니다. ;)
