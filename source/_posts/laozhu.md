---
title: 老朱
date: 2018-06-26 17:01:52
tags: [ test ]
published: false
---
# 老朱的屌炸了

* vue

```vue
<template>
     <div>
         <template v-for="item in list">
              <p>{{item.content}}</p>
              <img :src="item.img" alt="">
              <p class="divider"></p>
         </template>
     </div>
</template>

<script>
export default {
     data () {
         return {
             list : [
                 {content : 'ziksang',img :'https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=1301074775,1382810875&fm=80&w=179&h=119&img.JPEG'},
                 {content : 'ziksang2',img :'https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=1312092207,1376369244&fm=80&w=179&h=119&img.JPEG'}
             ]
         }
     }
}
</script>
<style>
body,html{
    width:100%;
    height:100%
}
.divider{
    width:100%;
    height:1px;
    background:black;
}
</style>
```