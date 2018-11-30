# 个人日常工作采坑成长记录

## 2018-11-03

1. `form`提交时，如果`input`框设置了`disabled`，那么不会提交该`input`框的值。这个以前真没意识到。。。网上说用`readonly`替代，但是我试了下并不符合我的预期，我加了一个`<input type="hidden" name="Data[type]" :value="tupe">`（vue）。

## 2018-11-04

1. `form`默认是`get`提交，我曾经看过，但是真的忘了，还是这两天的一个bug提醒了我。

## 2018-11-06

1. 接手的项目中有这么一段代码引发了兼容性问题：
    ```js
    $(this.$refs.tab).find("li[data-index=" + index + "]").get(0).scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'nearest'
    });
    ```
    一直用的chrome开发调试，没在意，直到测试在firefox上发现这里存在兼容性问题，随后去MDN上看`scrollIntoView`这个api，果然还是实验中的属性。firefox不支持"block" 设置项的值"nearest" 或 "center"，也不支持 "inline" 设置项。具体看：https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollIntoView

## 2018-11-12

1. 以前没用过vue的`slot`api，直到目前接手的项目中用到，才发现其实自己不怎么会，后来百度了下才算了解：https://laboo.top/2018/11/07/vue/

## 2018-11-17
> 其实是11.15号工作时了解到的内容，但由于当天通宵加班，今天才想起来。

1. 关于 Flexbox 布局（偏向各个属性实现原理讲解）：
  - https://github.com/xitu/gold-miner/blob/master/TODO1/flexbox-display-flex-container.md
  - https://github.com/xitu/gold-miner/blob/master/TODO1/flexbox-alignment.md
  
 2. <a href="https://github.com/xitu/gold-miner/blob/master/TODO1/adaptive-serving-using-javascript-and-the-network-information-api.md">使用 JavaScript 和网络信息 API 实现自适应服务</a>
 
 3. <a href="https://github.com/xitu/gold-miner/blob/master/TODO1/three-input-element-properties-that-i-discovered-while-reading-mdn.md">我在阅读 MDN 时发现的 3 个 Input 元素属性</a>
 
## 2018-11-20
今天遇到了个很不解问题，想了起码3小时愣是没想明白，是一个vue写的模拟table列表的组件，这里贴出部分代码：
```js
<template>
  <div class="panel-default">
    <div class="panel-head panelFixedWidth">
      <ul>
        <li class="toggle" >
          <label><input type="checkbox" v-model="checkAll"></label>
        </li>
        <li v-for="(item, index) in titleList" :key="index" :class="'wid-' + item.key">
          <select v-if="item.key==='MediaType'" v-model="adMediaType" @change="changeTitle" class="box-select">
            <option v-for="(opt, opt_index) of item.options" :key="opt_index" :value="opt.value" v-text="opt.title"></option>
          </select>
          <select v-else-if="item.key==='DayNums'" v-model="adDayNums" @change="changeTitle" class="box-select">
            <option v-for="(opt, opt_index) of item.options" :key="opt_index" :value="opt.value" v-text="opt.title"></option>
          </select>
          <select v-else-if="item.key==='IsActive'" v-model="adIsActive" class="box-select" disabled>
            <option v-for="(opt, opt_index) of item.options" :key="opt_index" :value="opt.value" v-text="opt.title"></option>
          </select>
          <span v-else v-text="item.title"></span>
        </li>
      </ul>
    </div>
    <div class="panel-body">
      <ul class="panel-content-ul">
        <li class="clearfix" v-for="(item, index) in dataList" :key="index" class="panelFixedWidth">
          <span class="toggle" >
            <label><input type="checkbox" :value="item.ID" v-model="checkList"></label>
          </span>
          <span 
            v-for="(item_value, item_key) in item" 
            :key="item_key + '-' + index" 
            :class="'wid-' + item_key" 
            :title="item_value" 
            v-if="item_key !== 'ID'"
            v-text="item_value">
          </span>
        </li>
      </ul>
    </div>
  </div>
</template>
```

> 省略了一些无关的代码，核心渲染代码就这么多。

- 问题是什么？
  - 如上面代码可见，这个模拟的table，表头和表中的数据我竟然不知道它如何保证一一对应的！！！
  - 起初我以为根据`key`来对应，但是也不对啊，vue中`key`应该是唯一的，而且表中的`key`后面都加上了`index`！这就很奇怪了，我也一度以为和后台约定好顺序它才能一一对应的，的确，在其他页面，后台返回的例如`BidMode`如果夹在三个`select`之中，那么列表渲染时数据没有一一对应，改变数据顺序就正常了，可是`Size`默认放在最后，而页面上`Size`却第4个展示，它还是一定程度上保证一一对应的！神烦，今天记录下来，明天继续搞！！！
  

## 2018-11-21
> 今天解决昨天遗留的困惑了。

1.首先，谷歌浏览器它接受后台传过来的对象时，会默认按照ascii码对对象键名进行排序，这就是为什么明明后台没有按顺序随意塞数据给我，而我接收时却总是看到对象键名类似英文字母排列。

2.针对昨天的问题

> 实在想不通，请了外援 **童萌（同事）**，他直接把数据在控制台打印出来，然后通过谷歌浏览器控制台右击选择 **Store as global variable**将其中一条数据保存为全局变量，然后对它进行`for in`遍历，然后震惊我了，遍历后的的顺序是这样的：

  ```js
    ID
    AdID
    AdName
    AdType
    Size
    BidMode
    MaterialType
    MediaType
    DayNums
    IsActive
  ```

  - 难怪`Size`这一列排在了第五，因为`for in`后它就是第五（`v-for`遍历对象时其实类似于`for in`）。这样只要保证表头数据与传过来的表中数据数量一致，字段一致，顺序无所谓，因为两个`v-for`保证了即使不同浏览器情况下它们遍历后的顺序也相同！！！
  - 这个问题的本质是`v-for`遍历后的顺序不是我在控制台看到的遍历前的顺序！！！ 
  - 昨天我不会右击选择 **Store as global variable** 这种方式，于是傻乎乎的在控制台直接`var obj = {...}`一个一个的写进去然后通过`for in`遍历，那时候发现顺序没变，就没想到`for in`上面，今天早上搜索发现当我们在控制台构造`Object`时，这时候遍历出来的键顺序与构造时一致，而引发上述问题的数据是后台传过来的，并不是我自己创建的。
  - 参考：http://w3help.org/zh-cn/causes/SJ9011
  
 
 ## 2018-11-22
 1.今天才知道css有个<a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/all">all</a>属性，除了`unicode-bidi`和`direction`都支持该属性！
 
 2.chrome调试技巧再度get！！！F12打开控制台，然后选择第一个`Elements`，再选中那个带箭头的图标，点击页面上的例如`input`框，下方选中`Event Listeners`(和Styles在同一行)即可知道该`input`上存在的事件以及相应的js文件！！！
 
 ## 2018-11-26
 1.今天老项目有个需求，就是在`select`下拉框中的每个`li`加个鼠标悬停显示文字的效果，一开始想用事件来做，后来发现直接加个`title`就能解决，省下一大笔代码。
 
 2.还有，`form`表单提交时，只要有`name`属性的都会提交数据，除了`disabled`。之所以记录这个，是因为一个页面在已经有了下一步按钮同时加上一个保存按钮，两个按钮都起到提交功能，但是跳转页面不一样，起初我想通过`callback`来做，代码较多，后来还是后台的哥们说直接在两个`button`上加不同的`name`属性，他根据这个来判断，果然好了，又省下一大段代码。汗颜。。。
   - 这里奇怪的是，明明两个`button`都加了`name`属性，怎么点击其中一个`button`貌似只传了被点击`button`的`value`值，另一个没被点击的居然没传，以前还真没注意到
 
## 2018-11-29
1.鼠标悬停加文字，最简单的方法就是给标签加个`title`属性。

2.`input`框在chrome里有个关于`autocomplete`问题，它会有个默认填充功能，这样输入框背景色会变成屎黄色，很难看，解决办法也较简单，`autocomplete=off`即可。

3.老项目中既用到了vue也用到了jq。开发时发现，如果css写在`<style scoped></style>`内，使用jq的`addClass`方法无法改变样式，需要将css写在非`scoped`style标签中才行。

4.`axios`请求下载报表功能，网上大同小异，但还是踩了坑：
  - 坑1：后端跟我说是get请求，还需要带参数，我想到了`axios.get()`方法，但是死活不行，百度发现人家这样写`axios({method: get, url: 拼接的url, responseType: 'blob'})`，但是由于我不想用这么low的拼接方法，然后用了官方提供的`axios.get(url, {params: 数据})`，但是乱码，百度说要加上`responseType:'blob'`，可我不知道在哪里加，最后试了下，下面写法就好了。
  ```js
  this.$http.get('downloadReport', {params: this.handleData, responseType:'blob'}).then((res) => {
    if (!res) {
      return
    }
    let fileName = res.headers['content-disposition'].split('=')[1]
    let url = window.URL.createObjectURL(res.data)
    let link = document.createElement('a')
    link.style.display = 'none'
    link.href = url
    link.setAttribute('download', `${fileName}`)
    document.body.appendChild(link)
    link.click()
  ```
  
  - 坑2：后端多做了处理，我给他的是数组，结果他当成对象来弄了。
  
## 2018-11-30
1.js遍历数组时需不需要将数组长度缓存起来？
  - 看：https://www.jianshu.com/p/486e9a6c6b80