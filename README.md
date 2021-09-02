# Auto Locale

## 痛点
一般的前端国际化都是预定义`{key:value}`的形式,如`{loginButton:'登录', start}`

```js
{
  Table: {
    filterTitle: 'Filter menu',
    filterConfirm: 'OK',
    filterReset: 'Reset',
    filterEmptyText: 'No filters',
    emptyText: 'No data',
    selectAll: 'Select current page',
    selectInvert: 'Invert current page',
    selectNone: 'Clear all data',
    selectionAll: 'Select all data',
    sortTitle: 'Sort',
    expand: 'Expand row',
    collapse: 'Collapse row',
    triggerDesc: 'Click to sort descending',
    triggerAsc: 'Click to sort ascending',
    cancelSort: 'Click to cancel sorting',
  },
  Modal: {
    okText: 'OK',
    cancelText: 'Cancel',
    justOkText: 'OK',
  },
  ……
}
```

组件代码还好，有确定的数量，维护起来也就那么点量

如果业务代码呢？需要国际化的文字是不可预估的。

如果是按以上方式去国际化业务代码如何？

可以

1. 但是成本非常高，这样会使业务代码的可读性极差，可以说是读起来非常难受

尤其是行业性质非常重的词汇，如 “整车成本分解”，“先期质量策划”，“采购归因分析”，其他新人来看代码怕是也比较吃力吧

2. 你来拿什么做key, 这个key怕是要想一会吧，谷歌翻译是免不了的吧。

3. 编码成本：边写代码边查阅国际化的定义，一种语言一个定义文件，写一些带固定文字的页面怕是要疯掉

4. 国际化英文还好，谁还不会一点英文呢？ 如果你的项目需要国际化日文呢，德文呢？ 一顿猛查吧？


### 为什么国际化的key必须是英文？
key为什么不可以是当前开发人员对应的国籍的语言文字?

用中文做key会有什么问题？


### 为什么机械式的工作不能交给机器？
耗时耗力机械工作还不如交给机器


### 大胆的想法

大胆的拿中文做key, 直观上好像离经叛道、违背常理、非常危险

其实不然

1. 拿中文做key,业务代码可读性强
2. 倾入性小，不用引入国际化文件，定义一个全局函数tr即可
3. 代码量小，不用做大量的国际化工作
4. 有机会实现全自动国际化，中文即是key,也是翻译的标的


### 全自动国际化？
可以

### 实现
使用nodejs通过正则匹配src下的所有tr('xxx'), 遍历文件夹获取所有的xxx, 以xxx做key，生成一个对象，然后把这个对象交给谷歌翻译api或者百度翻译api, 通过配置需要生成多种语言文件

### 不精确？
机器当然不能做到100%精确翻译，这个时候我们就要干预一下，自己手动维护一个customlocale文件夹，里面放置少量的重写覆盖的国际化配置文件即可

### 使用
1. 复制index.js的代码，放置在项目根目录scripts文件夹下，在代码前面的config配置中填上自己申请的百度翻译appid和key
标本语言：sampleLangulage
要生成的目标语言：langulages

```js

//配置：
const config = {
  //标本语言
  sampleLangulage: 'zh-CN',
  //要翻译的目标语言，key为antd所需的key
  langulages: {
    'en-US': '英语',
    'zh-CN': '中文简体',
    // 'ja-JP': '日语',
    // 'pt-BR': '葡萄牙语',
    // 'zh-TW': '中文繁体',
  },
  //百度翻译的appid和key
  baiduTranslate: {
    appid: '*******************',
    key:'*******************',
  }
};

```


2. 在package.json中添加

```json
{
  "scripts": {
    "locale": "node ./scripts/locale.js",
  }
}
```

3. 命令行中执行国际化脚本

```bash
npm run locale
```





### 注意
1. 代码里还维护了一个antd所需的国际化语言和百度翻译所需的国际化语言的key的转换字典
这个antdkey其实会生成以此命名的国际化文件,方便加载和使用
```js
//antd语言key和百度翻译语言key装换
function antdKeyToBaiduKey(antdkey) {
  let baidukey = antdkey;
  antdkey == 'zh-TW' && (baidukey = 'cht');
  antdkey == 'en-US' && (baidukey = 'en');
  antdkey == 'pt-BR' && (baidukey = 'pt');
  antdkey == 'ja-JP' && (baidukey = 'jp');
  return baidukey;
}
```

![image]('./snapshot.png')

2. 记得百度翻译好像是翻译服务申请一个月之后才能进行批量翻译，如果报错可能因为这个导致的，欢迎提issue