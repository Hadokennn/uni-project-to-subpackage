# uniapp2wxpack  
## Uni-App的微信小程序解耦构建，可以使整个uni项目作为一个包输出于其他小程序项目使用，可以自由的引入任何其他小程序的解耦包进行开发联调和构建  
  
### 对uni-app在微信小程序的打包方案进行改造，形成解耦打包，并且支持微信原生页面直接在uni-app项目中使用  
+ 可以使uni-app项目输出微信小程序的分包，被其他小程序项目使用  
  
+ 支持微信原生页面直接在uni-app项目中使用（还支持任何原生的js、wxss在uni-app项目中使用）  
  
+ 支持原生小程序项目直接在uni-app项目中进行开发，uni-app项目可以通过全局对象wx，在main.js或者App.vue中将相关的方法公开到wx对象中（因为解耦构建会在主小程序app.js的开头引入uni目录的app.js）  
  
+ 支持uni-app项目调用原生小程序项目中的资源   
  
+ 对uni包的App.vue的特殊处理方式（详见appMode）

### 快速上手  
#### 第一步  
准备一个uni-app项目  
  
#### 第二步  
在项目根目录，安装uniapp2wxpack  
````  
npx uniapp2wxpack --create
````  
  
#### 第三步 
配置根目录下projectToSubPackageConfig.js，可按需要替换主小程序目录中的内容  
  
#### 第四步  
运行开发命令
````  
npm run dev:mp-weixin-pack
````  
  
#### 第五步  
用微信开发者工具预览dist/dev/mp-weixin-pack目录
  
### 安装  
在已有的**uni-app**项目中通过cli安装  
````  
// 推荐  
npx uniapp2wxpack --create

// 或者全局安装uniapp2wxpack
npm i uniapp2wxpack -g
uniapp2wxpack --create
````  
+ 会在项目中创建projectToSubPackageConfig.js  
+ 创建mainWeixinMp目录（可根据projectToSubPackageConfig.js的配置修改目录名）  
+ 在package.json中会生成dev:mp-weixin-pack和build:mp-weixin-pack的命令  

### 概念 
+ uni-app项目目录   
    + project/src  
+ 主小程序项目（原生）目录  
    + project/mainWeixinMp   (可进行配置修改)  
+ uni-app项目中的原生小程序页面（或资源）目录  
    + project/src/wxresource   
+ uni-app项目打包输出之后在主小程序项目中的目录  
    + uniSubpackage (可进行配置修改)  

### 升级之后的脚手架结构 
与普通uni-app项目的结构保持一致
``````
│  babel.config.js  
│  package-lock.json 
│  package.json 
│  postcss.config.js
│  projectToSubPackageConfig.js             // 解耦包配置文件
│  README.md
├─dist
│  └─dev
│      ├─mp-weixin                          // uni-app微信小程序普通构建目录
│      └─mp-weixin-pack                     // uni-app微信小程序解耦构建目录（用于预览）
├─mainWeixinMp                              // 原生主小程序目录
├─public  
└─src                                       // uni-app源码目录
    │  App.vue
    │  LICENSE
    │  main.js
    │  manifest.json
    │  package.json
    │  pages.json
    │  README.md
    │  template.h5.html
    │  uni.scss
    │  
    ├─pages       
    ├─static
    ├─store
    ├─wxcomponents
    └─wxresource                            // 原生页面及资源存放目录
``````   
### 开发模式  
+ 单独开发解耦包  
在mainWeixinMp目录放置用于预览的原生小程序项目（hello world小程序即可），然后正常的在src中进行编码开发，解耦包打包完成后的包文件在dist/build/mp-weixin-pack/包名称  
  
+ 与完整小程序项目协同开发  
此时mainWeixinMp目录应该是真实的小程序项目（建议关联真实小程序项目的git仓库，mainWeixinMp作为一个git子仓库的存在），构建打包完成后即可将打包后内容(dist/build/mp-weixin-pack)覆盖mainWeixinMp的内容进行子仓库提交    

### 运行  
可根据实际项目情况修改以下两个命令的内容
````
// 开发
npm run dev:mp-weixin-pack

// 打包
npm run build:mp-weixin-pack
````  

### projectToSubPackageConfig.js   
解耦包配置文件  
````javascript
module.exports={
    // 微信原生小程序目录
    mainWeixinMpPath: 'mainWeixinMp',
    // uni项目输出的分包在微信原生小程序中的路径
    subPackagePath: 'uniSubpackage',
    // uni项目的App.vue中初始设置的处理方式，默认是relegation(降级模式)，[top(顶级模式) / none(丢弃)]
    appMode: 'relegation'
}
````   

### wxresource目录  
uni-app源码中要使用的原生页面及资源存放的目录  
wxresource目录中的页面都必须配置在pages.json的wxResource属性里  
**注意：wxresource目录构建后所在的物理路径，实际上就是src目录所在的路径，也就是uni包目录本身，所以构建后，wxresource中的文件和目录将被移动至uniSubpackage下，如果内容中有目录于src相同，则将会融合，目录名文件名都相同则将被丢弃**  
  
### appMode  
由于uni项目的App.vue对应小程序的app.js，在解耦包的模式中，会对主小程序的app.js产生冲突。  
所以设置了三种模式对App.vue进行处理，可以在projectToSubPackageConfig.js设置  
    
+ relegation (降级模式，默认)  
降级模式的含义是将uni目录从项目级别降级到了目录级别，那么App.vue中的钩子函数就只对目录有效  
比如：App.vue中的onLaunch，会降级成首次进入uni目录才触发，同样的，onHide会降级成只要离开uni目录的范围就会触发  
  
+ top (顶级模式)  
顶级模式的含义就是把uni的App.vue中的钩子和主小程序的app.js的钩子混合在一起  
**注意：顶级模式中uni包不能以分包形式存在，只能以主包的子目录形式存在（否则无法确保onLaunch的准确性），需要确保主小程序的app.js中引入了uni项目的app.js(自有项目会自动添加引入，如果是把构建解耦包提供给其他项目，需要其他项目的根目录下的app.js手动引入)**  
**如果不需要在根app.js中依赖uni包的方法和属性，可以不引用uni包的app.js，uni包依旧可以准确的触发onLauch和onShow**  
  
+ none (丢弃模式)  
丢弃模式就是不处理App.vue中的钩子  
#####  
降级模式和顶级模式都会将globalData和getApp()返回的内容进行混合  
所有模式都会将App.vue的初始设置保存在wx.__uniapp2wxpack.uniSubpackage.__packInit中（uniSubpackage属性是和uni的目录名保持一致的，如果修改目录名，属性名也会变更）  

**注意：如果要手动通过wx.__uniapp2wxpack触发App的钩子，需要首先确保触发onLaunch，否则App的onShow和onHide不会有效**  

### API  
+ wx.__uniapp2wxpack  
用于存放解耦包相关方法和数据的对象，在引入解耦包的app.js后，通过获取wx.__uniapp2wxpac.uniSubpackage.__packInit，可以拿到uni项目App.vue的初始化配置  
**注意：其中uniSubpackage属性代表了解耦包的名称，名称变化，该属性也会相应的改变**
  
+ __uniRequireWx  
只支持静态字符串参数  
在uni-app项目的源码目录中的vue、js文件需要引入原生的微信小程序资源（除了uni-app自带的wxcomponents目录外）都需要使用__uniRequireWx方法(类似node的require)，并且往往会配合目录别名@wxResource
````javascript
const nativeResource = __uniRequireWx('@wxResource/nativeJs/test')
const nativeExportDefaultObject = __uniRequireWx('@wxResource/nativeJs/test1').defaut
const {nativeRestObject} =  __uniRequireWx('@wxResource/nativeJs/test')
````  
+ __uniWxss  
只支持静态字符串参数  
在uni-app项目的源码目录中的vue、scss、less文件中引入原生的微信小程序wxss资源(类似@import 'xxxxxx'),往往会配合目录别名@wxResource  
````css
__uniWxss{
    import: '@wxResource/nativeWxss/1.wxss';
    import: '@wxResource/nativeWxss/2.wxss';
    import: '@wxResource/nativeWxss/3.wxss';
}
````
### @wxResource  
特殊的目录别名，此别名同时指向2个资源  
@wxResource只能在__uniRequireWx和__uniWxss中使用  
+ 指向src/wxresource  
+ 指向构建后的原生小程序项目中的uni解耦包目录  
#### 意味着src/wxresource会和uni解耦包融合构建  
````javascript  
// 跳出uni解耦包的目录，访问上层资源
__uniRequireWx('../@wxResource/top/1.js')
__uniRequireWx('@wxResource/../top/1.js')

// 绝对路径访问(访问原生小程序的根目录下的top/1.js)
__uniRequireWx('/top/1.js')
````  
### pack.config.js  
在构建完成的原生小程序项目中的uni解耦包目录下会存在pack.config.js，这个文件仅仅是保存了uni解耦包在主小程序中的目录名，以便解耦包中使用了动态路径（非相对路径）跳转页面或者加载图片地址  
注意：路径中保存的是绝对路径  
````javascript
const { packPath } = __uniRequireWx('@wxResource/pack.config.js')
uni.navigateTo({
    url: packPath + '/pages/about'
})
````  
### pages.json、主小程序的app.json混合处理  
+ 设置uni项目为分包  
需要在主小程序的app.json里配置分包的root，并且pages设置为[]  
构建时会把uni项目中的所有pages和subPackages中的pages都合并到预览目录中app.json的uni分包配置的pages里
````javascript
  // mainWeixinMp app.json
  "subPackages":[{
    "root":"uniSubpackage",
    "pages":[]
  }]
````  
+ 设置uni项目为主包并配置目录下的一些资源为分包  
在uni项目的pages.json里设置pages和subPackages  
+ 在uni项目中设置wxresource中的pages和subPackages  
需要在uni项目中的pages.json中的wxResource中配置pages和subPackages

### 解耦构建分包配置场景示例  
#### 场景一  
在uni项目中的vue页面分包  
pages.json  
````javascript
{
	// indexPage配置后，会提升为整个小程序项目的首页
	"indexPage":"",
	"pages": [
		{
			"path": "pages/test/about",
			"style": {
				"navigationBarTitleText": "测试"
			}
		}
	],
	"subPackages":[{
		"root": "pages/about",
		"pages": [{
			"path": "about",
			"style": {
				"navigationBarTitleText": "测试"
			}
		}]
	}],
	// uni项目中直接使用原生页面（src/wxresource/pages/test/index）
	// 由于wxresourece目录是个虚拟目录，所以路径中要去掉
	"wxResource":{
		"pages":["pages/test/index"]
	}
}
````  
#### 场景二  
在wxresource中使用分包  
pages.json
````javascript
{
	// indexPage配置后，会提升为整个小程序项目的首页
	"indexPage":"",
	"pages": [
		{
			"path": "pages/about/about",
			"style": {
				"navigationBarTitleText": "测试"
			}
		}
	],
	"wxResource":{
		"subPackages":[{
			"root": "pages/test",
			"pages": ["index"]
		}]
	}
}
````  
#### 场景三  
uni项目中vue页面和原生页面，在构建后同时在一个目录里，设置分包  
虽然设置在了不同的地方，但是构建后会合并到一起
````javascript
{
	// indexPage配置后，会提升为整个小程序项目的首页
	"indexPage":"",
	"pages": [
		{
			"path": "pages/about/about",
			"style": {
				"navigationBarTitleText": "测试"
			}
		}
	],
	// 构建后的目录uniSubpackage/pages/test
	"subPackages":[{
		"root": "pages/test",
		"pages": [{
			"path": "about",
			"style": {
				"navigationBarTitleText": "测试"
			}
		}]
	}],
	// 构建后的目录uniSubpackage/pages/test
	"wxResource":{
		"subPackages":[{
			"root": "pages/test",
			"pages": ["index"]
		}]
	}
}
````  
#### 场景四  
在小程序的app.json(mainWeixinMp/app.json)中将整个uniSubpackage配置成分包  
只需要对app.json进行配置即可，不需要修改pages.json中的subPackages和pages设置，构建后会将pages.json中所有的页面路径都抽取到uniSubpackage分包的配置里  
mainWeixinMp/app.json中的分包配置  
**注意：subPackages里的pages设为空数组**  
````javascript  
{
  "subPackages":[{
    "root":"uniSubpackage",
    "pages":[]
  }]
}
````


### 路径问题  
**uni项目中如果使用了绝对路径，在解耦构建的项目中，根路径是指向了主小程序的根的，所以需要自行拼接上uni解耦包的目录名，推荐使用pack.config.js中的packPath动态获取拼接**  
  
### 其他  
如果原生主小程序目录中已经存在了同uni解耦包命名相同的目录，在构建时，这个目录将被忽略，构建后的项目中的此目录是uni项目生成的解耦包
