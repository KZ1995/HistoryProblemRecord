## VUE专栏

## **问题一：webpack打包" cannot read property pop of undefined"问题指引**

### 一.基本信息

| ***node版本***   |   14.18.0   |
| ---------------- | :---------: |
| ***操作系统***   | win10专业版 |
| ***VUE版本***    |   3.2.33    |
| ***打包工具***   |   webpack   |
| ***包管理工具*** |    pnpm     |

### 二.解决过程

在完成某个功能模块后突然不能打包，报 cannot read property pop of undefined 错误，起初以为是某个包升级导致无法编译，因为可以本地运行，不过后面通过排查，在上一版本可以正常进行编译，那么问题回到代码端，使用最简单的排除法，通过还原git一个一个文件添加进去再进行分批打包，然后通过注释大法最终排查到以下代码出错

```javascript
import KVideo from '@/components/k-video'
```

为何引入组件会报错呢？百思不得其解于是继续查看组件内部文件

```javascript
<template>
  <div class="poster-box cursor-pointer flex justify-center items-center" :style="{width:width+'px',height:height+'px'}" @click="showPlay=true">
		<div class="mask"></div>
		<img :src="poster" v-if="poster" >
		 <el-icon :size="40" color="#fff" class="icon">
			 <el-icon-video-play/>
		 </el-icon>
	</div>
	<el-dialog v-bind="$attrs" v-model="showPlay">
		<sc-video :src="src" autoplay v-if="showPlay" :poster="poster"/>
	</el-dialog>
</template>

<script>
import ScVideo from '@/components/scVideo'

export default {
  name: "k-video",
	components:{ScVideo},
	props:{
		//视频路径
		src: { type: String, required: true, default: "" },
		//封面
		poster: { type: String, default: "" },
		//图片宽度
		width:{	type:[String,Number], default: "100"	},	
		//图片高度
		height:{	type: [String,Number], default: "100"	},
	},
	data() {
		return {
			showPlay: false
		}
	},
}
</script>

```

发现去除ScVideo这个包后打包恢复正常，但这个组件引用别处却是正常的，直到我发现框架自带的这个上传组件中的代码后恍然大悟

```javascript
import {defineAsyncComponent} from 'vue'
const scCropper = defineAsyncComponent(() => import('@/components/scCropper'))
```

 defineAsyncComponent参考vue[异步组件 | Vue.js (vuejs.org)](https://cn.vuejs.org/guide/components/async.html) 改用defineAsyncComponent引入后恢复正常

### 三.总结

该问题打包失败原因应该为解析代码时由于组件没有先行进行编译打包，ScVideo在注册后却没有引入文件，导致以上错误，可自行检查代码中是否是由于自定义组件或其它第三方组件初始化未编译导致，可以试试异步引入组件解决。