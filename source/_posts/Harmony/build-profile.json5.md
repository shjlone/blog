---
title: build-profile.json5
tags: Harmony 
toc: true
---

build-profile.json5文件分为工程级与模块级，其中buildOption在工程级文件和模块级文件均可配置，其中相同字段以模块级的字段为准，不同字段模块级的buildOption配置会继承工程级配置。

有点类似于Android中的build.gradle文件。

## 工程级示例

```json


{
  "app": { 
    //工程的签名信息，可包含多个签名信息
    "signingConfigs": [  
      {
        "name": "default",  //标识签名方案的名称，用户可自定义
        "type": "HarmonyOS",  //标识 HarmonyOS 应用
        //该方案的签名材料
        "material": {  
          "certpath": "D:\\SigningConfig\\debug_hos.cer",  //调试或发布证书文件，格式为.cer
          "storePassword": "******",  //密钥库密码，以密文形式呈现
          "keyAlias": "debugKey",  //密钥别名信息
          "keyPassword": "******",  //密钥密码，以密文形式呈现
          "profile": "D:\\SigningConfig\\debug_hos.p7b",  //调试或发布证书 Profile文件，格式为.p7b 
          "signAlg": "SHA256withECDSA",  //密钥库signAlg参数
          "storeFile": "D:\\SigningConfig\\debug_hos.p12"  //密钥库文件，格式为.p12
        },
        {
          "name": "config2",
        }
      }
    ],
    //定义构建的产品品类，如通用默认版、付费版、免费版等
    "products": [  
      {
        "name": "default",  //定义产品的名称，支持定制多product目标产物
        "signingConfig": "default",  //指定当前产品品类对应的签名信息，签名信息需要在signingConfigs中进行定义
        "compatibleSdkVersion": "5.0.0(12)",  //指定应用/服务兼容的最低版本
        "targetSdkVersion": "5.0.0(12)",  //指定应用/服务目标版本
        "runtimeOS": "HarmonyOS",  //指定为HarmonyOS
      }
    ],
    // 构建模式的集合,每个构建模式是指在执行不同target任务时使用何种构建配置的一套方案，默认打包hap时使用debug，打包app时使用release
    "buildModeSet": [
      {
        "name": "debug",   //定义构建模式的类型名称，系统默认给出test、debug和release，用户也可以自定义
        "buildOption": {   //配置项目在构建过程中使用的相关配置
          "packOptions": {  //包配置选项，可用于构建app时避免生成签名的hap
            "buildAppSkipSignHap": false
          },
          "debuggable": true,
          "resOptions": {
            "compression": {
              "media": {
                "enable": true // 是否对media图片启用纹理压缩
              },
              // 纹理压缩文件过滤，非必填，不填会压缩资源目录中的所有图片
              "filters": [
                {
                  "method": {
                    "type": "sut", // 转换类型
                    "blocks": "4x4" // 转换类型的扩展参数
                  },
                  // 指定用来参与压缩的文件，需要满足所有条件且不被exclude过滤的文件才会参与压缩
                  "files": {
                    "path": ["./**/*"], // 指定资源目录中的所有文件
                    "size": [[0, '10k']], // 指定大小10k以下的文件
                    // 指定分辨率小于2048*2048的图片
                    "resolution": [
                      [
                        { "width": 0, "height": 0 }, // 最小宽高
                        { "width": 2048, "height": 2048 } // 最大宽高
                      ]
                    ]
                  },
                  // 从files中剔除掉不需要压缩的文件，需要满足所有过滤条件的文件才会被剔除
                  "exclude": {
                    "path": ["./**/*.jpg"], // 过滤所有jpg文件
                    "size": [[0, '1k']], // 过滤大小1k以下的文件
                    // 过滤分辨率小于1024*1024的图片
                    "resolution": [
                      [
                        { "width": 0, "height": 0 }, // 最小宽高
                        { "width": 1024, "height": 1024 } // 最大宽高
                      ]
                    ]
                  }
                }
              ]
            }
          },
          //cpp相关编译配置
          "externalNativeOptions": {
            "path": "./entry/src/main/cpp/CMakeLists.txt",  //CMake配置文件，提供CMake构建脚本
            "arguments": [],  //传递给CMake的可选编译参数
            "abiFilters": [  //HarmonyOS当前支持的ABI编译环境
              "arm64-v8a",
              "x86_64"
            ],
            "cppFlags": ""  //设置C++编译器的可选参数
          },
          "sourceOption": {   //使用不同的标签对源代码进行分类，以便在构建过程中对不同的源代码进行不同的处理
            "workers": []
          },
          //配置筛选har依赖.so资源文件的过滤规则
          "nativeLib": {
             "filter": {
               //按照.so文件的优先级顺序，打包最高优先级的.so文件
               "pickFirsts": [
                 "**/1.so"
               ],
               //按照.so文件的优先级顺序，打包最低优先级的.so文件
               "pickLasts": [
                 "**/2.so"
               ],
               //根据正则表达式排除匹配到的.so文件，匹配到的so文件将不会被打包
               "excludes": [
                 "**/3.so", //排除所有名称为“3”的so文件
                 "**/x86_64/*.so" //排除所有x86_64架构的so文件
               ],
               //允许当.so重名冲突时，使用高优先级的.so文件覆盖低优先级的.so文件
               "enableOverride": true
            }
          },
        }
      }   
    ]
  },
  "modules": [
    {
      "name": "entry",  //模块名称，须与模块中module.json5文件中的module.name保持一致
      "srcPath": "./entry",  //标明模块根目录相对工程根目录的相对路径
      "targets": [  //定义构建的APP产物，由product和各模块定义的targets共同定义
        {
          "name": "default",  //target名称，由各个模块的build-profile.json5中的targets字段定义
          "applyToProducts": [  
            "default"   //表示将该模块下的"default" Target打包到"default" Product中
          ]
        }
      ]
    }
  ]
}

```

## 模块级示例

```json

{
  "apiType": "stageMode",  //API类型，支持FA(faMode) 和 Stage(stageMode)模型
  "showInServiceCenter": true,  //是否在服务中心展示
  "buildOption": {  //配置项目在构建过程中使用的相关配置
    //配置筛选har依赖.so资源文件的过滤规则
    "nativeLib": {
       "filter": {
          //按照.so文件的优先级顺序，打包最高优先级的.so文件
          "pickFirsts": [
             "**/1.so"
           ],
           //按照.so文件的优先级顺序，打包最低优先级的.so文件
           "pickLasts": [
              "**/2.so"
           ],
           //根据正则表达式排除匹配到的.so文件，匹配到的so文件将不会被打包
           "excludes": [
              "**/3.so", //排除所有名称为“3”的so文件
              "**/x86_64/*.so" //排除所有x86_64架构的so文件
           ],
           //允许当.so重名冲突时，使用高优先级的.so文件覆盖低优先级的.so文件
           "enableOverride": true
       }
    },
    "sourceOption": {   //使用不同的标签对源代码进行分类，以便在构建过程中对不同的源代码进行不同的处理
      "workers": []
    },
    //cpp相关编译配置
    "externalNativeOptions": {
      "path": "./src/main/cpp/CMakeLists.txt",  //CMake配置文件，提供CMake构建脚本
      "arguments": [],  //传递给CMake的可选编译参数
      "abiFilters": [  //HarmonyOS当前支持的ABI编译环境
        "arm64-v8a",
        "x86_64"
      ],
      "cppFlags": ""  //设置C++编译器的可选参数
    },
    //ArkTS编译配置
    "arkOptions":{
      "types":[]  //配置d.ts/d.ets的相对路径或包名，用于使用自定义的声明类型
    },
  },
  "buildOptionSet": [  //buildOption的集合，
    {
      "name": "release",  //定义buildOption的名字，取值有default、debug 和 release，也可自定义
      "arkOptions": {
        "obfuscation": {  //针对release模式下的配置
          "ruleOptions": {
            "enable": false,   // 5.0.3.600及以上版本，默认为false
            "files": [   //混淆文件的相对路径
              "./obfuscation-rules.txt"
            ]
          },
          "consumerFiles": './consumer-rules.txt' //仅Static Library模块可配置：默认导出的混淆规则
        }
      },
      "debuggable": true,  //定义编译模式是否为debug
      "copyFrom": "release",  //从指定的buildOption中复制相关配置
      "resOptions": {
        "compression": {
          "media": {
            "enable": true // 是否对media图片启用纹理压缩
          },
          // 纹理压缩文件过滤，非必填，不填会压缩资源目录中的所有图片
          "filters": [
            {
              "method": {
                "type": "sut", // 转换类型
                "blocks": "4x4" // 转换类型的扩展参数
              },
              // 指定用来参与压缩的文件，需要满足所有条件且不被exclude过滤的文件才会参与压缩
              "files": {
                "path": ["./**/*"], // 指定当前模块资源目录中的所有文件
                "size": [[0, '10k']], // 指定大小10k以下的文件
                // 指定分辨率小于2048*2048的图片
                "resolution": [
                  [
                    { "width": 0, "height": 0 }, // 最小宽高
                    { "width": 2048, "height": 2048 } // 最大宽高
                  ]
                ]
              },
              // 从files中剔除掉不需要压缩的文件，需要满足所有过滤条件的文件才会被剔除
              "exclude": {
                "path": ["./**/*.jpg"], // 过滤所有jpg文件
                "size": [[0, '1k']], // 过滤大小1k以下的文件
                // 过滤分辨率小于1024*1024的图片
                "resolution": [
                  [
                    { "width": 0, "height": 0 }, // 最小宽高
                    { "width": 1024, "height": 1024 } // 最大宽高
                  ]
                ]
              }
            }
          ]
        }
      },
      //cpp相关编译配置
      "externalNativeOptions": {
        "path": "./src/main/cpp/CMakeLists.txt",  //CMake配置文件，提供CMake构建脚本
        "arguments": [],  //传递给CMake的可选编译参数
        "abiFilters": [  //HarmonyOS当前支持的ABI编译环境
          "arm64-v8a",
          "x86_64"
        ],
        "cppFlags": ""  //设置C++编译器的可选参数
      },
      "sourceOption": {   //使用不同的标签对源代码进行分类，以便在构建过程中对不同的源代码进行不同的处理
        "workers": []
      },
      //配置筛选har依赖.so资源文件的过滤规则
      "nativeLib": {
          "filter": {
            //按照.so文件的优先级顺序，打包最高优先级的.so文件
            "pickFirsts": [
              "**/1.so"
            ],
            //按照.so文件的优先级顺序，打包最低优先级的.so文件
            "pickLasts": [
              "**/2.so"
            ],
            //根据正则表达式排除匹配到的.so文件，匹配到的so文件将不会被打包
            "excludes": [
              "**/3.so"
            ],
            //允许当.so重名冲突时，使用高优先级的.so文件覆盖低优先级的.so文件
            "enableOverride": true
         }
      },
    }
  ],
  "buildModeBinder": [   //构建模式与构建配置的关联配置,通过该配置可以将不同的构建配置和target进行组合，并绑定到对应的构建模式上,其中构建模式需要在工程级别的构建模式列表中
    {
      "buildModeName": "debug",
      "mappings": [   //构建模式绑定中的具体映射表，描述的是target和构建配置的一对一的关系
        {
          "targetName": "default",
          "buildOptionName": "release"
        }
      ]
    }
  ],
  "targets": [  //定义的target，开发者可以定制不同的target，具体请参考配置多目标构建产物章节
    {
      "name": "default",
    },
    {
      "name": "ohosTest",
    }    
  ]
}

```


## useNormalizedOHMUrl



## 参考

- [https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-hvigor-build-profile-V5](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-hvigor-build-profile-V5)
