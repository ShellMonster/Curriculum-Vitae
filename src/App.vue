<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 40,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* Inspired by https://www.geekaso.com/
* 你好，我是道长，
* 一个ASO优化师，半个运营，
* 半个程序员，半个分析师，
* 非常有幸能在这里，
* 跟你分享我的个人简介，
*/

/* 我先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景好像太单调了，来点背景吧 */
html {
  color: rgb(222,222,222); background: rgb(0,43,54);
}
/* 文字是不是离边框太近了？ */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* 做个代码高亮看看 */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 再加点 3D 效果呗 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* 接下来我给自己准备了一个编辑器 */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 好了，我开始写简介了 */


`,
          `
/* 这个简历好像差点什么
 * 对了，这是 Markdown 格式的，我需要变成对 浏览者 更友好的格式
 * 简单，用开源工具翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `道长@ASO黑科技
----

一个ASO优化师，半个运营，
半个程序员，半个分析师。

个人博客 [GeekASO](https://www.geekaso.com) 。
个人博客 [ASO黑科技](https://www.asoheikeji.com) 。

职业经历
----

* 2014-2015，贵州铜仁，趣分期本校内负责人，曾带领团队做到单月单个校区业绩全国Top3；
* 2015-2016，浙江杭州，分期管家运营实习，曾搭建100+新媒体公号矩阵，自然引流；
* 2016-2017，浙江杭州，分期管家ASO运营，负责金融马甲包上架，投放，于2017年率先分享[ivp6](https://blog.csdn.net/ShellM/article/details/72621427)及[4.3马甲](https://blog.csdn.net/ShellM/article/details/72293253)解决方案；
* 2017-2019，浙江杭州，傲熠网络SO运营，负责金融马甲包上架，投放、负责公司机房搭建维护，测试投放策略；并于期间承接部分金融马甲包上架，投放优化等服务；
* 2019.9-2019.11，北京，巨掌数据分析师，负责ASO研究，数据分析，公司ASO技能培训；研究出App高覆盖策略，1000-22000覆盖，10000-35000覆盖；
* 2019.11-，北京，七麦数据分析师，负责ASO研究，数据分析，公司ASO技能培训。

个人技能
----

1. ASO优化，
2. Python自动化，
3. Shell搬砖命令，
4. 数据分析，

课外实践
----

1. 三节课增长黑客P1助教
2. 三节课数据科学P1助教
3. 三节课电商运营助教

其他
----

* 微信小程序：**ASO黑科技Pro**
* 微信公众号：**ASO黑科技**

> 如果你喜欢这个简介效果，Fork [这个项目](https://github.com/jirengu-inc/animating-resume)吧，也可以打造你自己的个人简介！

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
