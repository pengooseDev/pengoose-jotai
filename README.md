# @pengoose/jotai

<div align="center">

<h3>A simple and powerful state convention manager for React using Jotai.</h3>
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
  <h4> * Illustration created by <a href="https://twitter.com/takuan1469414">沢庵</a> </h4>

[English](./README.md) | [한국어](./docs/README_KO.md) | [日本語](./docs/README_JA.md)

</div>

## Introduction

`@pengoose/jotai` is a `Convention manager` for managing state in React applications using Jotai. It provides a simple and structured way to define and manage your application's state, making it easier to organize and maintain your state logic. By following a set of conventions, you can create a consistent and scalable state management system that is easy to understand and maintain.

## Installation

Install the package using npm:

```sh
npm install @pengoose/jotai
```

Or using yarn:

```sh
yarn add @pengoose/jotai
```

## Getting Started (Step by Step)

1. Define the interfaces of the states you want to manage using atoms.
2. Inject the interfaces into the generic type of the `AtomManager`.
3. Create a class that manages the state by inheriting the `AtomManager` abstract class.

### Step1: Define your state interfaces to manage with atoms

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

### Step2: Create an AtomManager class

- Extend the `AtomManager` class to create your state manager.
- you have to implement selectors and actions in the `AtomManager` class.

```ts
import { atom } from 'jotai';
import { AtomManager } from '@pengoose/jotai';
import { PlaylistStatus, Music } from '@/types';

// 1. Extend AtomManager to create your state manager
export class Playlist extends AtomManager<PlaylistStatus> {
  // 2. Implement selectors
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

  // 3. Implement actions
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

  // 4. Implement helper methods
  private isEmpty(playlist: Music[]) {
    return playlist.length === 0;
  }

  private isFirstMusic(index: number) {
    return index === 0;
  }
}

// 5. Create an instance of the Playlist class
const initialState: PlaylistStatus = {
  playlist: [],
  index: 0,
};

export const playlistManager = new Playlist(initialState);
```

---

### Step3: Wrap instance of AtomManager with `useManager` hook

- The `useManager` hook wraps the instance of the `AtomManager` class and converts the abstracted selectors and actions into `useAtomValue` and `useSetAtom` hooks, respectively, inferring the types for the user.

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

### ⛳️ If you Don't use `useManager` hook 🥲

- You can use Jotai's `useAtomValue` and `useSetAtom` hooks to get and set the state of the `AtomManager` instance without using `useManager`. However, it is a bit cumbersome to use. 😨

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

### Step4: Use custom hook in your components 🚀

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

## Summary

The `AtomManager` class is designed to be used with custom hooks to encapsulate the state management logic and make it easier to use in your components.

> Flow: Class(AtomManager) --> custom hook --> Component(View)

<div align="center">
  😗👍
</div>

## Contributing

Contributions are welcome! For major changes, please open an issue first to discuss what you would like to change. ;)
