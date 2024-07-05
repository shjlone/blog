---
title: tsconfig.json
tags: TypeScript
---


如果一个目录下存在 tsconfig.json 文件，那么它意味着这个目录是 TypeScript 项目的根目录。



```json
// configs/base.json
{
  "compilerOptions": {}
}

// tsconfig.json

{
  "extends": "./configs/base", //继承配置
  "compilerOptions": { //编译选项
    "incremental": true, //开启增量编译
    "tsBuildInfoFile":"./",//指定增量比编译的文件位置
    "diagnostics":true,//打印诊断信息
    "listFiles":true,//打印编译的文件
    "":"",
    /* 指定编译版本： 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */
    "target": "es6",
    /* 指定模块代码的生成方式: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
    "module": "commonjs", //模版标准
    "lib": [], //指定要包含在编译中的库文件
    "allowJs": true, //允许编译js文件
    "checkJs": true, //检查和报告js文件中的错误
    "jsx": "preserve",//指定jsx代码用于的开发环境： preserve、react、react-native
    "declaration": false, //是否在编译的时候生成相应的d.ts声明文件。declaration和allowJs不能同时设置为true
    "declarationMap": true, //是否在编译的时候生成相应的map声明文件
    "sourceMap": true,//
    "outFile": "./", //指定输出文件合并为一个文件，只有设置module的值为amd和system模块时才支持这个配置
    "outDir": "./", //指定输出文件夹
    "rootDir": "./",//指定编译文件的根目录
    "composite": true,//是否编译构建引用项目
    "removeComments": true,//是否将编译后的文件注释删除掉 
    "noEmit": true,//不生成编译文件
    "importHelpers": true,//是否引入tslib里的复制工具函数，默认为false
    "downlevelIteration": true,//当target为“es5”或“es3”时，为“for-of”、“spread”和“destructuring”中的迭代器提供完全支持
    "strict": true,//开启所有类型检查
    "noImplicitAny": true,//没有设置明确类型会报错
    "strictNullChecks": true, //为true时，null和undefined值不能赋值给非这两种类型的值，别的类型的值也不能赋给他们，除了any类型，还有个例外就是undefined可以赋值给void类型
    "baseUrl": "/",//设置解析非相对模块名称的基本目录，
    "paths": {//基于baseUrl的路径映射
      "":"",
    },
    /* 指定模块的解析策略： node、classic */
    /* 若未指定，那么在使用了 --module AMD | System | ES2015 时的默认值为 Classic，其它情况时则为 Node */
    "moduleResolution": "",
    "": "",
  },
  /* 指定需要编译的单个文件列表 */
  "files": [

  ],
  "include": [], //指定要编译的路径列表
  "exclude": [],//要排除的不编译的文件/文件夹列表


}
```


## 参考

- [耗时一年整理，全网最全的TypeScript踩坑集锦(tsconfig.json 常用配置项注释以及问题）](https://wuzaofeng.github.io/learning/typescript/ts-collection-config.html#tsconfig-json-%E5%B8%B8%E7%94%A8%E9%85%8D%E7%BD%AE%E9%A1%B9)
- [https://www.tslang.cn/docs/handbook/tsconfig-json.html](https://www.tslang.cn/docs/handbook/tsconfig-json.html)
- [https://wangdoc.com/typescript/tsconfig.json](https://wangdoc.com/typescript/tsconfig.json)