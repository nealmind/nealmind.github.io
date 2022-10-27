---
layout: post
title: 标题
tags: 标签
author: neal

---
* content
{:toc}
# 2022-10-26-init-vue-sample



## Vue3 api

* 文档地址：https://cn.vuejs.org/guide/essentials/application.html#mounting-the-app



## 组件使用：

`@` 绑定事件，eg：```<button @click="handleClick">添加</button>```

* 定义应用：

``` vue
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
	const { createApp } = Vue
        const appSample = createApp({
            data() {
                return {
                    message: 'vue sample :',
                    inputInfo: '',
                    listData: [],
                }
            },
            methods: {
                handleClick() {
                    this.listData.push(this.inputInfo)
                    this.inputInfo = ''
                }
            }
        })

        appSample.component('todo-item', {
            props: ['itemsData'],
            template: '<li>{{itemsData}}</li>'
        })	
        appSample.mount('#appTest')
</script>		
```



`props` 属性：

HTML 中`attribute` 属性不支持驼峰命名，上例中 itemsData 在引用时按 items-data 识别