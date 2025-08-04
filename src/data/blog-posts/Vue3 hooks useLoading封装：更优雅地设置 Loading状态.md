
---
 title: "Vue3 hooks useLoading封装：更优雅地设置 Loading状态"
 slug: "vue3-hooks-useloading"
 publishDate: "2023-10-20"
 description: "本文介绍一个简单而有效的 Vue 3 Hook——useLoading，它能够帮助开发者轻松管理异步请求的加载状态。"
---

### 简介

在现代前端开发中，处理异步请求的加载状态是提升用户体验的重要环节。用户在等待数据加载时，提供一个清晰的反馈可以显著减少不确定性和焦虑感。本文将介绍一个简单而有效的 Vue 3 Hook——`useLoading`，它能够帮助开发者轻松管理加载状态。

###  背景介绍

在单页应用（SPA）中，用户与应用的交互通常涉及到多个异步请求，例如 API 调用、数据获取等。为了提升用户体验，开发者需要在请求发出时显示加载状态，并在请求完成后隐藏加载状态。传统的做法通常需要在每个请求中手动管理加载状态，这样不仅繁琐，而且容易出错。

`useLoading` Hook 的出现正是为了解决这个问题。它通过封装加载状态管理的逻辑，使得开发者可以专注于业务逻辑，而无需重复编写加载状态的管理代码。

而我们此次 `useLoading` 封装，不仅使用方便，而且适配 传统 `Promise` 处理以及 `async/await` 使用模式，更加灵活方便。

### 使用指南

##### 安装
确保你已经在项目中安装了 Vue 3。然后可以直接将 `useLoading.ts` 文件放入你的 hooks 目录中。

#####  示例代码
在你的 Vue 组件中使用 `useLoading` Hook，如下所示：

```ts
<template>
    <div v-loading="loading"></div>
</template>

<script lang='ts' setup>
import { useLoading } from '@/hooks';
import { DataApi } from '@/api'; // 假设你有一个数据 API

const { loading, loadingWrapper } = useLoading();

/**
* 请求数据的方法 async/await
*/
const queryDataList = async () => {
    const res = await loadingWrapper(DataApi.queryData({})
    // ... 处理数据
}

/**
* 请求数据的方法 Promise.then
*/
const queryDataList = () => {
    loadingWrapper(DataApi.queryData({}).then(res => {
        let data = res.data.data.list;
        // ... 处理数据
    }));
}

</script>
```

有朋友可能会问，我同一个文件中有多个请求怎么办，不用担心，忘了 ts/js 支持

```ts
<template>
  <div v-loading="loading"></div>
  <div v-loading="loadingDelete"></div>
</template>

<script lang="ts" setup>
import { useLoading } from "@/hooks";
import { DataApiAdd, DataApiDelete } from "@/api"; // 假设你有一个数据 API
/**
 * 多个请求
 * 请求数据的方法 async/await
 */
const { loading, loadingWrapper } = useLoading();

const queryDataAdd = async () => {
  const res = await loadingWrapper(DataApiAdd.queryData({}));
  // ... 处理数据
};

const { loading: loadingDelete, loadingWrapper: loadingWrapperDelete } = useLoading();

const queryDataDelete = async () => {
  const res = await loadingWrapperDelete(DataApiDelete.queryData({}));
  // ... 处理数据
};
</script>
```

### 源码

#### 完整代码

```ts
import { ref, computed } from "vue";
export default function useLoading() {
  /**
   * 使用number类型的loadingIndex
   * 更适用于多个请求的场景
   */
  const loadingIndex = ref(0);
  /**
   * 计算属性，根据loadingIndex的值判断
   * @returns boolean
   */
  //
  const loading = computed(() => {
    return loadingIndex.value <= 0 ? false : true;
  });
  // 进入loading，loadingIndex的值加1
  const enterLoading = () => {
    loadingIndex.value += 1;
  };
  /**
   * 退出loading，loadingIndex的值减1
   */
  const quitLoading = () => {
    if (loadingIndex.value > 0) {
      loadingIndex.value -= 1;
    } else {
      console.warn("no need quit loading");
    }
  };

  /**
   * 包装请求，自动完成进入/退出loading的设置
   * @param promise - 需要包装的Promise
   * @returns 返回包装后的Promise
   */
  const loadingWrapper = <T>(promise: Promise<T>) => {
    return new Promise<T>((resolve, reject) => {
      // 进入loading
      enterLoading();
      promise
        .then((result) => {
          resolve(result);
        })
        .catch((error) => {
          reject(error);
        })
        .finally(() => {
          // 退出loading
          quitLoading();
        });
    });
  };
  return {
    loading,
    enterLoading,
    quitLoading,
    loadingWrapper,
  };
}
```
##### 适用范围
`useLoading` Hook 适用于需要对多个异步请求进行加载状态管理的场景。它可以用于：

- API 数据请求
- 文件上传/下载
- 任何需要异步操作的场景

## 总结

`useLoading` Hook 是一个简单而强大的工具，可以帮助开发者高效地管理加载状态，提升用户体验。通过封装复杂的状态管理逻辑，开发者可以专注于业务逻辑，从而提高开发效率。希望本文能帮助你更好地理解和使用 `useLoading` Hook！
