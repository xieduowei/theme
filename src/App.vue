<template>
  <div id="app">
    <div class="color-change">
      <div>
		  <input type="color" v-model="primaryColor"  name="color" id="color" > 
	  </div>
      <div class="color-change1">我要变颜色1</div>
      <div class="color-change2">我要变颜色2</div>
      <div class="color-change3">我要变颜色3</div>
    </div>
    <div>
      <!-- <button @click="changeColor('#ffffff')">变吧</button> -->
    </div>
  </div>
</template>

<script>
	import Color from 'css-color-function'
	import themeConfig from './js/themeConfig.js'
export default {
  name: 'App',
  data(){
	  return {
		  primaryColor:'#ff571a'
	  }
  },
  watch:{
	  primaryColor:function(val,oldval){
		  this.changeColor(val,oldval)
	  }
  },
  methods:{
	selectColor(){
		
	},
	generateColors(primary){
		return themeConfig.map(f => {
		    const value = f.exp.replace(/primary/g, primary);  // 将字符串中的primary 关键字替换为实际值，以便下一步调用 `Color.convert`
		    return Color.convert(value);     // 生成一连串的颜色值，可以看见计算值全部变为了`rgb/rgba` 值
		});
	  },
    changeColor(color,oldColor){
		let oldColorCluster = [oldColor,...this.generateColors(oldColor)];
		let newColorCluster = [color,...this.generateColors(color)]
		console.log('我是老颜色',oldColorCluster)
        // 拿到所有初始值之后，因为我们要做的是字符串替换，所以这里利用了正则
        this.initStyleReg = oldColorCluster  
            .join('|')
            .replace(/\(/g, '\\(') // 括号的转义
            .replace(/\)/g, '\\)')
            .replace(/0\./g, '.');  // 这里替换是因为默认的css中计算出来的值透明度会缺省0，所以索性就直接全部去掉0
		console.log('我是新颜色',newColorCluster)
		console.log('我是正则',this.initStyleReg)
		let styles = document.querySelectorAll('.ui-theme');
		
		if(styles.length > 0){
			styles = Array.from(styles) // 有标记就找标记的,节省性能开销
		}
		else{
			styles = Array.from(document.querySelectorAll('style')).filter(style=>{ // 找到需要进行替换的style标签
				let text = style.innerText;
				let re = new RegExp(`${this.initStyleReg}`, 'i');  // i为模糊大小写匹配
				return re.test(text);
			})
		}
		const re = new RegExp(`${this.initStyleReg}`, 'ig');// ig 全局匹配
		
		
		styles.forEach(style => {
			const { innerText } = style;
			style.innerHTML = innerText.replace(re, match => {
				//查找颜色为rgb的索引
				let index = oldColorCluster.indexOf(match.toLowerCase().replace('.', '0.'));
				//没找到rgb的,就再查找颜色为RGB的索引,
				if (index === -1){
					index = oldColorCluster.indexOf(match.toUpperCase().replace('.', '0.'));
				}
				// 进行替换
				return newColorCluster[index].toLowerCase().replace(/0\./g, '.');
			});
			//添加标记,方便下次查找
			style.setAttribute('class', 'ui-theme');
		});
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.color-change1{
	color:rgb(255, 128, 82)
}
.color-change2{
	color:rgb(77, 20, 0)
}
.color-change3{
	color:rgb(255, 236, 229)
}
</style>
