
## 基础知识


目前官方推荐的最佳解决方案，是官方对于Navigation导航组件的封装，使用更简单便捷。如果熟悉Navigation的话，使用起来很快上手。


**首先先集成HMRouter模块**
使用命令行安装依赖：



```
ohpm install @hadss/hmrouter

```

或在模块的 oh\-package.json5 文件中添加依赖



```
{
  "dependencies": {
    "@hadss/hmrouter": "^1.0.0-rc.6",
  }
}

```

插件配置
在项目级的 hvigor/hvigor\-config.json5 中添加插件依赖



```
{
  "dependencies": {
    "@hadss/hmrouter-plugin": "^1.0.0-rc.7"
  },
}

```

在模块的 hvigorfile.ts 文件中应用插件



```
import { hapTasks } from '@ohos/hvigor-ohos-plugin';
import { hapPlugin } from '@hadss/hmrouter-plugin';  //导入插件

export default {
    system: hapTasks,
    plugins:[hapPlugin()]   //应用插件，
}

//还有harPlugin(),hspPlugin(),模块是什么类型，就用对应的方法

```

在项目的build\-profile.json5中，配置useNormalizedOHMUrl属性为true



```
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "debug",
        "compatibleSdkVersion": "5.0.0(12)",
        "runtimeOS": "HarmonyOS",
        "buildOption": {
          "strictMode": {
            "useNormalizedOHMUrl": true
          }
        }
      }
    ],
  }
}

```

HMRouter的配置就完成了。


**初始化HMRouter**


在入口的 UIAbility 类中（通常名字为EntryAbility ）的 onCreate() 方法中调用HMRouter初始化方法，



```
export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, 'testTag', '%{public}s', 'Ability onCreate');
    //初始化HMRouterMgr  
    HMRouterMgr.init({
      context: this.context
    })
  }
}

```

**定义首页**


HMNavigation 用来包裹首页内容：
其中 homePageUrl 为加载时显示的页面地址，options中可以设置框架的属性（NavModifier）和首页的元素，设置内容基本与Navigation相同。



```
class NavModifier extends AttributeUpdater<NavigationAttribute> {
  initializeModifier(instance: NavigationAttribute): void {
    instance.mode(NavigationMode.Stack);
    instance.navBarWidth('100%');
    instance.titleMode(NavigationTitleMode.Mini)
    instance.hideBackButton(true)
    // instance.hideNavBar(true)
  }
}


@Entry
@Component
struct HMRouterPage {
  static readonly TAG = 'HMRouterPage'
  modifier: NavModifier = new NavModifier()

  aboutToAppear(): void {

    HMRouterMgr.registerPageBuilder({
      builder: wrapBuilder(PageOneBuilder),
      pageUrl: "pageOne",
    })
  }

  build() {
    Column() {
      HMNavigation({
        homePageUrl: '',
        navigationId: 'mainNavigation', options: {
          // standardAnimator: HMDefaultGlobalAnimator.STANDARD_ANIMATOR,
          // dialogAnimator: HMDefaultGlobalAnimator.DIALOG_ANIMATOR,
          modifier: this.modifier,
          title: { titleValue: "首页" },
          menus: [{
            value: "menu1",
            icon: "resources/base/media/menu1.png",
            action: () => {

            }
          }, {
            value: "menu2",
            icon: "resources/base/media/menu2.png",
            action: () => {

            }
          }]
        }
      }) {
        PageMain()
      }
    }.width('100%').height('100%')
  }
}

```

**定义子页**
使用@HMRouter定义子页面路由
使用HMRouterMgr的 push()，replace()，pop()等方法进行页面之间的导航
使用HMRouterMgr.getCurrentParam()获取当页面的参数



```
//PageA
@HMRouter({ pageUrl: 'pageA' })
@Component
export struct PageA {
   build() {
    Column() {
      Text("Page A")
      Button("Page B").onClick(() => {
        HMRouterMgr.push({ pageUrl: 'pageB', param: "param" })
      })
      Button("Back").onClick(() => {
        HMRouterMgr.pop({ param: "pageA backParam" })
      })
    }
  }
}

//PageB
@HMRouter({ pageUrl: 'pageB'})
@Component
export struct PageB {
  @State params: string | undefined = HMRouterMgr.getCurrentParam()?.toString()
 
  build() {
    Column(){
      Text("Page B")
      Text("params:" + this.params)
      Button("Page B").onClick(()=>{
        HMRouterMgr.pop({ param: "pageB backParam" })
      })
    }
  }
}

```

另一种方法HMRouterMgr.registerPageBuilder
使用HMRouterMgr.registerPageBuilder()动态注册子页面路由，子页面内容使用NavDestination组件包括，无缝将Navigation框架迁移到HMRouterMgr



```
//定义子页面builder
@Builder
export function PageOneBuilder() {
  PageOne()
}

@Component
export struct PageOne {
  @State params: string | undefined = HMRouterMgr.getCurrentParam()?.toString()

  build() {
    NavDestination() {
      Column() {
        Text("page1")
          .padding(10)
          .fontSize(20)
          .fontWeight(500)
      }
      .width('100%')
      .height('100%')
      .alignItems(HorizontalAlign.Center)
      .justifyContent(FlexAlign.Center)
    }
    .title("page1")
    .menus([{
      value: "menu1",
      icon: "resources/base/media/menu1.png",
      action: () => {

      }
    }, {
      value: "menu2",
      icon: "resources/base/media/menu2.png",
      action: () => {
      }
    }])
  }
}

//动态注册路由
HMRouterMgr.registerPageBuilder({
  builder: wrapBuilder(PageOneBuilder),
  pageUrl: "page1",
})

```

HMRouter还有拦截器，生命周期，转场动画，服务路由等功能，限于篇幅不在细说，可以查看官方文档：
[HMRouter官方文档](https://github.com)
下面我们就开始搭建练习项目的路由框架


## 项目实践


**注意：原本项目实战中打算使用HMRouter组件，实际发现使用HMRouter时菜单icon无法动态更新，因此替换成Navigation组件。**
我们的demo中涉及三个页面，首页，详情页，作者页。首页作为路由主页面，详情页及作者页作为路由子页面。因此我们只需要注册后2个页面路由即可。


**首页**



```
@Builder
function PageMap(name: string) {
  if (name == PageURls.PoetryDetailPage) {
    PoetryDetailPage()
  } else if (name == PageURls.AuthorPage) {
    AuthorPage()
  }
}


//首页
@Entry
@Component
struct Index {
  @Provide('navPathStack') navPathStack: NavPathStack = new NavPathStack()

  build() {
    Stack() {
      Navigation(this.navPathStack) {
        Column() {
          Button(PageURls.PoetryDetailPage)
            .onClick(() => {
              this.navPathStack.pushPath({ name: PageURls.PoetryDetailPage })
            })
        }
      }
      .navDestination(PageMap)
      .titleMode(NavigationTitleMode.Mini)
      .title("首页")
      .hideBackButton(true)
    }.width('100%').height('100%')
  }
}

```

**详情页**



```
//详情页
@Component
export struct PoetryDetailPage {
  @Consume('navPathStack') navPathStack: NavPathStack

  build() {
    NavDestination() {
      Text("详情页")
    }.title({
      main: this.poetry!!.title,
      sub: `[${this.poetry!!.dynasty}]${this.poetry!!.author}`
    }).menus([{
      value: "作者",
      icon: "resources/base/media/ic_person_24px.svg",
      action: () => {
        this.navPathStack.pushPath({ name: PageURls.AuthorPage })
      }
    }])
  }
}

```

**作者页**



```
//作者页
@Component
export struct AuthorPage {
    @Consume('navPathStack') navPathStack: NavPathStack

  build() {
    NavDestination() {
      Column() {
        Text("AuthorPage")
      }
      .alignItems(HorizontalAlign.Center)
      .justifyContent(FlexAlign.Center)
      .width('100%')
      .height('100%')
    }.title(this.name)
  }
}

```


```
//PageUrl
export class PageUrls{
  public static PoetryDetailPage = 'PoetryDetailPage'
  public static AuthorPage = 'AuthorPage'
}

```

到此我们基本的导航框架已经搭建完成，下一步我们来实现首页的列表展示。




---


本文的技术设计和实现都是基于作者工作中的经验总结，如有错误，请留言指正，谢谢。


 本博客参考[slower加速器](https://jisuanqi.org)。转载请注明出处！
