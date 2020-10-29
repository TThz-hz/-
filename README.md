# [前端随记](#)
* [CDN加速](#CDN加速)
* [vue跨域配置](#vue跨域配置)
* [umi-request](#umi-request)
* [富文本 tinymce 本地引用](#富文本tinymce本地引用)
* [vue动画初体验](#vue动画初体验)
### CDN加速
vue.config.js
```js
/** 
 *  开发环境时使用本地 npm 包
 *  发布环境时使用 CDN 包
 */
  chainWebpack(config) {
    config.when(process.env.NODE_ENV === 'development',config=> {
       config.entry('app').clear().add('./src/main.js')/* 开发环境入口文件 */
    })
    config.when(process.env.NODE_ENV !== 'development', config => {
        config.entry('app').clear().add('./src/main-prod.js')/* 生产环境入口文件 */
        config.set('externals',{
            vue: 'Vue',
            'vue-router': 'VueRouter',
            echarts: 'echarts',
            nprogress: 'NProgress',
            'driver.js': 'Driver',
            html2canvas: 'html2canvas',
            VueCropper: 'vue-cropper',
            // ElementUI默认导出 ELEMENT 使用全局变量 ELEMENT 即可
       })
     }
  }
```
main-prod.js
```js
import Vue from 'vue'
/** 
 *  ElementUI默认导出 ELEMENT，
 *  采用CDN加速时，无需引入element-ui，直接使用全局变量 ELEMENT
 */
Vue.use(ELEMENT, { size: 'small', zIndex: 10001 })
```
index.html
```html
<!--!!! 版本号最好与package-lock.json文件中的版本号一致 ！！！-->
<% if (process.env.VUE_APP_BASE_URL /* 判断是否为生产环境 */ ) { %>
    <link crossorigin="anonymous" href="https://unpkg.com/element-ui@2.13.2/lib/theme-chalk/index.css" rel="stylesheet">
    <!--<link crossorigin="anonymous" href="https://unpkg.zhimg.com/element-ui@2.13.2/lib/theme-chalk/index.css" rel="stylesheet">-->
    <link crossorigin="anonymous" href="//cdn.staticfile.org/driver.js/0.9.8/driver.min.css" rel="stylesheet">

    <script crossorigin="anonymous" src="https://cdn.staticfile.org/vue/2.6.10/vue.min.js"></script>
    <script crossorigin="anonymous" src="https://cdn.staticfile.org/vue-router/3.0.6/vue-router.min.js"></script>
<!-- 暂时没有找到免费好用的 element-ui CDN地址  -->
    <script crossorigin="anonymous" src="https://unpkg.com/element-ui@2.13.2/lib/index.js"></script>
    <!--<script crossorigin="anonymous" src="https://unpkg.zhimg.com/element-ui@2.13.2/lib/index.js"></script>-->
    <!--<script crossorigin="anonymous" integrity="sha384-WbhdtWslh0AUD1Dhf8OExUvvjZ/VN6o2HHMsYlDXb6uf3IweMH13dGL4V/KgDc7y" src="https://lib.baomitu.com/element-ui/2.13.2/index.js"></script>-->
    <!--<script src="https://cdnjs.cloudflare.com/ajax/libs/element-ui/2.13.2/index.js" integrity="sha512-5ISWXWibAAxzJ+/xjZh0r6yM+fVNH2RpMWSN1DrPlonE4Ll7pHIM79Wv2aNt+6+RgKp6OIpVeFakPRON1XRXqg==" crossorigin="anonymous"></script>-->

    <script crossorigin="anonymous" src="https://cdn.staticfile.org/driver.js/0.9.8/driver.min.js"></script>
    <script crossorigin="anonymous" src="https://cdn.staticfile.org/nprogress/0.2.0/nprogress.min.js"></script>
    <script crossorigin="anonymous" src="https://cdn.jsdelivr.net/npm/echarts@4.9.0/dist/echarts.min.js"></script>
    <script crossorigin="anonymous" src="https://cdn.jsdelivr.net/npm/html2canvas@1.0.0-rc.7/dist/html2canvas.min.js"></script>
    <script crossorigin="anonymous" src="https://cdn.jsdelivr.net/npm/vue-cropper@0.5.5/dist/index.min.js"></script>
<% } %>
```
### vue跨域配置
vue.config.js
```js
  devServer: {
		proxy: {
			'/proxy': {
				target: "http://www.xxxxxx.com/",    // 代理地址
				changeOrigin: true,
				pathRewrite: {
					'^/proxy': '' // 路径重写
				}
			},
		},
  },
```
### npm run build-test
跟路径下新增文件 .env.prod 与 .env.deve
```
NODE_ENV = 'production'
# .env.prod 发布环境 base api
VUE_APP_BASE_URL = 'http://www.生产环境.com/api/'
```
```
NODE_ENV = 'production'
# .env.deve 测试环境 base api
VUE_APP_BASE_URL = 'http://www.测试环境.com/api/'
```
package.json
```json
  "scripts": {
    "dev": "vue-cli-service serve",
    "build": "vue-cli-service build --mode prod",   // 生产环境
    "build-test": "vue-cli-service build --mode deve",   // 测试环境
  },
```
### umi-request
src/api/index.js
```js
import request from 'umi-request';
import router from '@/router'

let Loading,Message;
if (process.env.VUE_APP_BASE_URL) { /* 生产环境 */
    Loading = ELEMENT.Loading;
 	Message = ELEMENT.Message;
}else { /* 开发环境 */
 	Loading = require('element-ui').Loading;
 	Message = require('element-ui').Message;
}

let loadinginstace // 全局加载loading
let timer = null; // 防抖

/**
 * 网络error提示
 * @param {String} text 错误信息
 */
function errorMessage(text, bool=false) {
  let callNow = !timer;
	timer && clearTimeout(timer);
  timer = null;
	timer = setTimeout(() => { timer = null }, 500);
	callNow && Message({ message: text, type: 'error', duration: 2500, offset: 80 })
	bool && router.replace({ path: '/login' })
}

/**
 * 实例中间件 { global: false }
 * @param {Object} ctx 上下文对象，包括 req 和 res 对象
 * @param {Function} next 调用下一个中间件的函数
*/
request.use(async (ctx, next) => {
	loadinginstace = Loading.service({
	    fullscreen: true,
	    background: "rgba(0, 0, 0, 0.8)",
	    text:"拼命加载中",
	    spinner: 'el-icon-loading'
	});
	await next();
	loadinginstace.close?.();
	// const { res } = ctx;
	// res?.error_code === 1001 ? errorMessage(res?.mess || '登录已过期，请重新登录',true) :
	// res?.error_code !== 0 && errorMessage(res?.mess || '请求失败');
})

/**
 * HTTP状态码
*/
const codeMessage = {
  // 200: '请求成功。',
  // 201: '新建或修改数据成功。',
  // 202: '一个请求已经进入后台排队（异步任务）。',
  // 204: '服务器成功处理，但未返回内容。',
  400: 'status：400，请求错误，服务器没有进行操作。',
  401: 'status：401，用户没有权限（身份认证错误）。',
  403: 'status：403，服务器收到的请求，但拒绝执行此请求。',
  404: 'status：404，请求路径不存在，服务器没有进行操作。',
  406: 'status：406，请求的格式错误。',
  410: 'status：410，请求的资源被永久删除。',
  422: 'status：422，验证错误。',
  500: 'status：500，服务器错误，无法完成请求。',
  502: 'status：502，网关错误。',
  503: 'status：503，服务器暂时过载或维护。',
  504: 'status：504，网关超时。',
};

/**
 * POST请求-统一处理响应信息
 * @param {String} url 请求地址
 * @param {Object} params 请求参数
 * @param {Number} timeout 超时时间(15000)
 */
export function POST(url, params, timeout=15000) {
	return new Promise((resolve, reject) => {
		request.post(url, {
			headers: { "Content-Type": "application/json" },
			credentials: 'include', // 默认请求是否带上cookie
			timeout,
			prefix: process.env.NODE_ENV === 'development' ? '/proxy/api/' : process.env.VUE_APP_BASE_URL,
			data: process.env.NODE_ENV === 'development' ? { ...params, publick: '公共参数' } : { ...params },
			errorHandler: err=> {
				loadinginstace.close?.();
				console.warn({
					Error:err?.stack || '未知错误',
					ErrorInfo: {
						request: err.request || {},
						response: err.response || {}
					}
				})
				const { response = {} } = err;
				const errortext = codeMessage[response?.status] || '网络出错，请检查网络后重试';
				errorMessage(errortext);
			}
		}).then(res=> {
			res.error_code == 1001 ? errorMessage(res.mess || '登录已过期，请重新登录',true) :
				String(res.error_code) !== '0' && errorMessage(res?.mess || '请求失败');
			String(res.error_code) === '0' ? resolve(res) : reject(res)
		}).catch(err=> { reject(err) })
	})
}

/**
 * POST请求-单独处理响应信息
 * @param {String} url 请求地址
 * @param {Object} params 请求参数
 */
export function POSTrequest(url, params) {
	return request.post(url, {
		headers: { "Content-Type": "application/json" },
		credentials: 'include', // 默认请求是否带上cookie
		timeout: 10000,
		prefix: process.env.NODE_ENV === 'development' ? '/proxy/api/' : process.env.VUE_APP_BASE_URL,
		data: process.env.NODE_ENV === 'development' ? { ...params, publick: '公共参数' } : { ...params },
		errorHandler: err=> {
			loadinginstace.close?.();
			console.warn({
				Error:err?.stack || '未知错误',
				ErrorInfo: {
					request: err.request || {},
					response: err.response || {}
				}
			})
		}
	})
}
```

### 富文本tinymce本地引用
```shell
// version:3.2.3 
$ npm i @tinymce/tinymce-vue 
```
先正常使用不做任何配置，将请求的js文件保存到本地路径
![image](https://s1.ax1x.com/2020/10/29/BGLdc6.png)
![image](https://s1.ax1x.com/2020/10/29/BGO6qU.png)

src/compontents/Tinymce/index.vue
```html
<template>
  <div >
  <!-- 富文本 https://www.tiny.cloud/my-account/onboarding/ --> 
    <tinymce-editor ref="editor" :init="init" :disabled="disabled" v-model="myValue" @onKeyUp="()=>{this.$emit('change', this.myValue)}" />
  </div>
</template>

<script>
import Editor from "@tinymce/tinymce-vue";
import './tinymce.min.js'

export default {
  name: 'Tinymce',
  components: { "tinymce-editor": Editor },
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    width: { default: "100%" },
    height: { default: "76vh" },
    value: {
      type: String,
      default: ""
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default: "link lists image code table print preview pagebreak nonbreaking hr emoticons directionality charmap advlist formatpainter"
    },
    toolbar: {
      type: [String, Array],
      default: `undo redo print preview formatpainter pagebreak hr 
                bold italic underline strikethrough alignleft aligncenter alignright alignjustify 
                forecolor backcolor superscript subscript bullist numlist indent outdent blockquote nonbreaking emoticons ltr rtl charmap 
                link unlink image table removeformat code formatselect fontsizeselect `
    }
  },
  watch: {
    value(val){
      this.myValue = val
    }
  },
  data() {
    return {
      imgList: [],
      myValue: this.value,
      init: {
        theme: 'silver',
        language: "zh_CN",
        //替换为本地路径 有条件的可以替换为自己的 CDN 地址
        language_url: process.env.VUE_APP_BASE_URL ? "/project/tinymce/langs/zh_CN.js" : "/tinymce/langs/zh_CN.js",
        theme_url: process.env.VUE_APP_BASE_URL ? "/project/tinymce/themes/silver/theme.min.js" : "/tinymce/themes/silver/theme.min.js" ,
        base_url: process.env.VUE_APP_BASE_URL ? '/project/tinymce' : '/tinymce',
        height: this.height,
        width: this.width,
        plugins: this.plugins, //插件
        toolbar: this.toolbar, //工具栏
        toolbar_mode: 'wrap',
        zIndex: 1101,
        branding: false,
        images_upload_handler:(blobInfo, success,failure)=> { // 上传图片
          let file = blobInfo.blob()
          if (file.size/1024/1024 > 20) {
            this.$message.error('此图片大小超过20M,请压缩后上传')
            failure('此图片大小超过20M,请压缩后上传');
            return false
          }
          let base64Img = `data:${file.type};base64,`+blobInfo.base64()
          this.$POST('AuploadWareDetailsImg', {base64Img}).then(res=> {
            success(res.ObjectURL)
            this.imgList.push({imgUrl: res.ObjectURL, imgId: res.imgId})
          }).catch(err=>{
            failure('上传图片失败');
          })
        }
      },
    }
  },
}
</script>

::v-deep .tox-notifications-container {
  display: none !important;
}

<!-- 使用 -->
components: { Tinymce: ()=> import('@/components/Tinymce') },
<Tinymce width="100%" height="76vh" v-model="value" />
```

### vue动画初体验
需求如下：实现类似加入购物车的动画，点击孙组件中按钮将小球移动到 子组件<Navbar />中的按钮处
![image](https://i.bmp.ovh/imgs/2020/10/786ad2ab9a615026.png)
![image](https://ftp.bmp.ovh/imgs/2020/10/72c84681d5064955.gif)
实现思路：从点击事件event中获取点击坐标，将小球从点击位置移到<Navbar />中的按钮位置

组件件的通信采用 provide + inject 的方式

---
父组件Layout中：
```html
<navbar ref="navbar" />

<script>
	provide(){
		return {
			_CPN: this.$refs
		}
	},
</script>
```

子组件NavBar中：
```html
<!-- 小球最终到达的位置 -->
<el-button id="task_queue" >看我</el-button>

<!-- 小球 -->
<div class="ball_wrap">
  <transition 
    @before-enter="beforeEnter"
    @enter="enter"
    @afterEnter="afterEnter"
  >
  <!-- Y轴非线性动画 -->
  <div class="ball" v-show="ball.show" :style="{ 'top': y+'px', 'left': x+'px' }">
    <!-- X轴线性动画 -->
    <div class="inner"></div>
  </div>
  </transition>
</div>

<script>
/**
 * 获取元素在DOM文档中的位置(
 * @param {Object} element DOM元素
 * @return {Object} { top, left }
 */
function getElementPosition(element) {
  let top = element.offsetTop; 
  let left = element.offsetLeft;
  let current = element.offsetParent;
  while (current !== null) {
    top += current.offsetTop;
    left += current.offsetLeft;
    current = current.offsetParent;
  }
  return {top,left}; 
}

  data() {
    return {
      ball:{
        show: false,
        event: null
      }
    }
  },
  methods: {
    startTransition(event){
      let {clientX, clientY} = event // 获取点击坐标
      let {top, left} = getElementPosition(document.getElementById('task_queue'))
      // 获取按钮坐标（因为浏览器窗口的大小会发生变化，所以要重新设置小球的位置）
      this.x = left + 74
      this.y = top - 7
      this.ball.event = {clientX, clientY}
      this.ball.show = true
    },
    beforeEnter(el){
      let x = this.x - this.ball.event.clientX + 7
      let y = this.ball.event.clientY - this.y - 7
      el.style.display = ''
      let inner = el.querySelector('.inner');
      el.style.webkitTransform = `translate3d(0, ${y}px, 0)`;
      el.style.transform = `translate3d( 0, ${y}px, 0)`
      //处理内层动画
      inner.style.webkitTransform = `translate3d( -${x}px, 0, 0)`;
      inner.style.transform = `translate3d( -${x}px, 0, 0)`;
    },
    enter(el, done) {
      this._offset = document.body.offsetHeight // 触发重绘
      let inner = el.querySelector('.inner');
      el.style.webkitTransform = `translate3d(0, 0, 0)`
      el.style.transform = `translate3d(0, 0, 0)`
      inner.style.webkitTransform = `translate3d( 0, 0, 0)`;
      inner.style.transform = `translate3d( 0, 0, 0)`;
      el.addEventListener('transitionend', done)
    },
    afterEnter(el) {
      this.ball.show = false
      this.ball.event = null
      el.style.display = 'none'
    },
  }
</script>

<style>
.ball_wrap {
  .ball {
    position: fixed;
    z-index: 10001;
    transition: all 1s cubic-bezier(0.49, -0.49, 0.75, 0.7);
    .inner {
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background-color: #f56c6c;
      transition: all 1s linear;
    }
  }
}
</style>
```
  
孙组件Index中：
```html
  <el-button @click="btnCilck($event)" />
  inject: ['_CPN'],
  btnCilck(event){
      const { navbar } = this._CPN // 拿到Navbar组件实例
      navbar?.startTransition?.(event) // 将点击事件event传给Navbar组件的startTransition方法
  }
```
