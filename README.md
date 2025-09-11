# 小程序转Taro工具

官方@tarojs/cli-convertor的修复版本，主要修复以下问题：

## 一、taroize is not a function

### 故障原因

官方提交[统一各包的构建产物目录为 dist](https://github.com/NervJS/taro/pull/17910/files#diff-6eed60ac39958d7bb19f49d5670760827bcc1ee6202ee29d490629c17a11001e)，遗漏了一块地方：`import * as taroize from '@tarojs/taroize'`，需要将其修改成`import parse from '@tarojs/taroize'`，这是具体 [diff地址](https://github.com/NervJS/taro/pull/17910/files#diff-6eed60ac39958d7bb19f49d5670760827bcc1ee6202ee29d490629c17a11001e)。

### 解决方案

将文件 `packages/taro-cli-convertor/src/index.ts` 中的 `import * as taroize from '@tarojs/taroize'`，修改成 `import parse from '@tarojs/taroize'`。

## 二、Scope#references is not available in Babel 8. Use Scope#referencesSet instead

### 故障原因

Babel官方在 7.26.10 以上的版本加入了 Babel 8，导致在npx运行过程中，运行到了 Babel 8，而 `cli-convertor` 又没锁定版本 `"@babel/traverse": "^7.24.1""`，最终安装的是7.28.4的@babel/traverse，导致的问题。

```
function resetScope(scope: Scope) {
  if (!process.env.BABEL_8_BREAKING) {
    // @ts-expect-error(Babel 7 vs Babel 8)
    scope.references = Object.create(null);
    // @ts-expect-error(Babel 7 vs Babel 8)
    scope.uids = Object.create(null);
  } else if (scope.path.type === "Program") {
    scope.referencesSet = new Set();
    scope.uidsSet = new Set();
  }

  scope.bindings = Object.create(null);
  scope.globals = Object.create(null);
}
get references() {
  throw new Error(
      "Scope#references is not available in Babel 8. Use Scope#referencesSet instead.",
  );
}
```

### 解决方案

给 `cli-convertor` 的 `@babel/traverse` 锁定版本：`"@babel/traverse": "~7.26.10"`。