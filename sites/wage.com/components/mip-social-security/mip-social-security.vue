<template>
  <div class="wrapper">
    <div class="">
      <label for="">
        <slot/>
      </label>
      <div
        :class="{open:isOpen}"
        class="onOff"
        @click="closeOrShow"><span
          :class="{active:isOpen}"
          class="onOff_off"/></div>
    </div>
    <div v-show="isOnClick">
      <div
        v-show="isOpen"
        :scrollshow="scrollshow"
        :class="{'ip-disable':isAfter}"
        class="arrow"
        @click="openScrollSelect()">
        <label for="">缴纳标准</label>
        <input
          :value="selectValue"
          :class="{'c-disable':cok}"
          type="text"
          class="ip-disable">
      </div>	
    </div>
    
    <div
      v-show="isOpen"
      class="wage">
      <label for="">缴纳基数</label>
      <input
        :class="{'ip-disable':ok}"
        :value="getBase"
        type="number"
        placeholder="0-10,000,000最多两位小数"
        @input="inputBaseMoney">
    </div>
    <!-- <span>{{tag}}</span> -->
    <!--  <div v-show="isshow"> -->
    <div
      v-show="isOpen && isshow"
      class="wage wage2 arrow arrow2">
      <label for="">缴纳比例</label>
      <input
        type="number"
        placeholder="请输入5-12"
        class="w96p"
        @input="inputProportion"
      >
    </div>
    <mip-fixed
      v-show="scrollshow"
      type="bottom"
      class="container z1001">
      <mip-scroll-select
        :scrollshow="scrollshow"
        :values="showValues"
        :tag = "getTag"
        class="container z1001"
        @showselect="showSelect"
        @colsescrollselect="colseScrollSelect"/>
    </mip-fixed>

  </div>
</template>
<script>
export default {
	props: {
		isshow: {
			type: Boolean,
			default: false
		},
		fromwage: {
			type: Number,
			default: 0
		},
		tag: {
			type: Number,
			default: 1
		},
		isclick:{
			type:Number,
			default:0,
		},
		socialsecurity: {
			type: Object,
			default: () => {
				return {
					aged: {
						baseMoney: 3384,
						maxMoney: 25401,
						proportion: 8
					},
					noWork: {
						baseMoney: 3387,
						maxMoney: 25401,
						proportion: 0.2
					},
					medical: {
						baseMoney: 5080,
						maxMoney: 25401,
						proportion: 2
					},
					disease: {
						baseMoney: 0,
						proportion: 0
					},
					accumulation: {
						baseMoney: 2273,
						maxMoney: 25401
					}
				};
			}
		}
	},
	data () {
		return {
			isOpen: true,
			security: {},
			baseMoneys: {},
			scrollshow: false,
			selectValue: '按照工资',
			ok: true,
			baseMoney: '',
			fromWage: true,
			fromBase: '',
			isLowst: {
			},
			amount: '',
			numberAmount: '',
			isAfter: false,
			cok:false,
			isOnClick:true,
			wage:0,
		};
	},
	computed: {
		getBase:{
			get: function() {
				if (this.fromWage) {
					return this.fromwage;
				} else {
					if (this.isLowst.socialLowst === 1) {
						return this.socialsecurity.noWork.baseMoney;
					}
				}
			}
		},
		showValues:{
			get: function () {
				return	this.showValuesF();
			}
		},
		getTag : {
			get: function(){
				return this.isclick;
			}
		}
	},
	methods: {
		showValuesF: function(){
			if (this.isshow) { // 公积金
				if (this.tag === 1) { // 税前
					this.selectValue = '按照工资';
					this.fromWage = true;
					this.ok = true;
					this.cok = false;
					this.isAfter = false;
					this.isOpen?this.isOnClick = true:'';
					return ['按照工资', '自定义'];
				} else { // 税后
					this.selectValue = '自定义';
					this.fromWage = false;
					this.ok = false;
					this.cok = true;
					this.fromBase = 0;
					this.isAfter = true;
					this.isOnClick = false;
					return ['自定义'];
				}
			} else { // 社保
				this.isAfter = false;
				if (this.tag === 1) { // 税前
					this.selectValue = '按照工资';
					this.fromWage = true;
					this.ok = true;
					this.cok = false;
					this.isOpen?this.isOnClick = true:'';
					return [
						'按照工资',
						'按照最低标准',
						'自定义'
					];
				} else { // 税后
					this.selectValue = '自定义';
					this.fromWage = false;
					this.ok = false;
					this.cok = false;
					this.fromBase = 0;
					this.baseMoney = 0;
					this.isOpen?this.isOnClick = true:'';
					return [
						'自定义',
						'按照最低标准'
					];
				}
			}
		},
		closeOrShow: function () {
			this.isOpen = !this.isOpen;
			!this.isOpen?this.isOnClick = false:'';
			if (this.isshow) { // 公积金
				// this.accumulationSelect = this.isOpen;
				this.security.accumulationSelect = this.isOpen;
			} else { // 社保
				this.security.socialSelect = this.isOpen;
			}
			this.$emit('securityselect', this.security);
		},
		openScrollSelect: function () {
			this.scrollshow = true;
		},
		showSelect: function (obj) {
			this.fromWage = false;
			this.isLowst = {};
			this.$emit('getselectvalue', 2);
			if (obj.detail[0] != '' && obj.detail[0] != '0') {
				this.selectValue = obj.detail[0];
			}
			if (this.selectValue == '自定义') {
				// 基数可编辑
				this.ok = false;
				this.baseMoney = '';
				this.fromBase = '';
				this.isshow ? this.isLowst.accumulationLowst = 3 : this.isLowst.socialLowst = 3;
				this.$emit('islowst', this.isLowst);
				this.fromWage = false;
			} else {
				if (this.selectValue == '按照工资') {
					this.$emit('getselectvalue', 1);
					this.fromWage = true;
					this.baseMoney = this.fromwage;
					this.isshow ? this.isLowst.accumulationLowst = 2 : this.isLowst.socialLowst = 2;
					this.$emit('islowst', this.isLowst);
				} else { // 最低标准
					this.fromWage = false;
					if (!this.isshow) { // 社保
						this.fromBase = this.socialsecurity.noWork.baseMoney;
						this.baseMoney = this.socialsecurity.noWork.baseMoney;
						this.isLowst.socialLowst = 1;
						this.$emit('islowst', this.isLowst);
					}
				}
				this.ok = true;
			}
			this.scrollshow = false;
			this.sendBaseMoney();
		},
		colseScrollSelect () {
			this.scrollshow = false;
		},
		// 公积金比例
		inputProportion: function (e) {
			if(e.target.value){
				let sNum = e.target.value.toString(); // 先转换成字符串类型
				if (sNum.indexOf('.') > 0) {
					let index = sNum.indexOf('.');
					sNum = sNum.slice(0, index);
				}
				e.target.value = sNum;
				this.$emit('getproportion', e.target.value);

			}

		},
		inputBaseMoney: function (e) {
			if(e.target.value){
				let match = e.target.value.match(/^\d{1,8}\.\d{0,2}|\d{1,8}/);
				const max = 10000000;
				if (match[0] > max) {
					//double的有效位数是10,小数位分了两位,故整数位限制为8位
					(match[0]/max) >= 10&&(match[0]/max) <=20?match[0] = max:match[0] = match[0].substring(0, 7);

				}
				if(match){
					e.target.value = match[0];
					this.wage = match[0];
				}
			}else{
				this.wage&&this.wage.length>1?e.target.value = this.wage:e.target.value = '';
			}
			this.baseMoney = Number(e.target.value);
			this.fromBase = this.baseMoney;
			this.sendBaseMoney();
			// if(e.target.value){
			// 	this.baseMoney = Number(this.moneyFilter(e.target.value));
			// 	e.target.value = this.baseMoney;
			// 	this.fromBase = this.baseMoney;
			// 	this.sendBaseMoney();
			// }
		},
		moneyFilter(str) {
			//过滤掉非法字符(也会过滤掉',')
			var items = str.match(/\d+|\.|。/g);
			var num;
			if (items == null) {
				return '';
			} else {
				num = items.join('');
			}
			var result = '',
				dec = '';
			num = num.replace(/。/g, '.'); //兼容中文输入法
			var list = num.split('.');
			const max = 10000000;
			if (list[0] > max) {
				//double的有效位数是10,小数位分了两位,故整数位限制为8位
				(list[0]/max) >= 10&&(list[0]/max) <=20?result = list[0].substring(0, 8):result = list[0].substring(0, 7);
			} else {
				result = list[0];
			}
			if (list.length > 1) {
				//小数限制为两位
				if (list[1].length > 2) {
					dec = list[1].substring(0, 2);
				} else {
					dec = list[1];
				}
				if (dec.length > 0) {
					//加上小数
					result < 10000000 ? result = result + '.' + dec : '';
				}
			}
			return result;
		},
		sendBaseMoney: function () {
			let baseMoneys = {};
			this.isshow ? (baseMoneys.accumulationMoney = this.baseMoney) : (baseMoneys.socialMoney = this.baseMoney);
			this.$emit('getsecuritybase', baseMoneys);
		}
	}
};
</script>

<style scoped>
div{
	-webkit-tap-highlight-color: transparent; 
}
.arrow2{
	background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAAUCAYAAACJfM0wAAABe0lEQVQ4T52UYVUDQQyEpxKwQCUUCZyEIoFKoBJAQpEAEqgEKgEkUAnt+97LHOmyu70jv+52s5NJMslCv3YraZ3+3yV9p//a54ck3t1JOmaHRfwAuJN0E2A447iRRICa3UsCGJ/X0gFgwL7iYpB0kLSKRxwvSzbh6zfc/zGAnyQ9S3qRtE0enHHHGXfZHiNDiOxbwJQAx4cibcrzFmmSbjbYUn+AqwZj6kS9yuiuIYwygDNssiUSwHMYux8EI8Omza2x2dKwrhTnqAIJfkYfypqjIgKNWraO3WVSI03qi2WNUjIayjCYLcrhLSXyW1R0MDCHvcnjDiVk6VmODBADAmtKhS0zcK8XKIeHHhYY/gTzPCDuwXYKsGWX2Vrj5VARnD7spwB70VSZVabyRCbXgM2sXDRjyg3gi+bVakzDkBBKyNYa93Fae4x7iyZvxLz9PMXN5vGQJvQWjcvBmkVySBIy/A8txpMWTQABhhooGQFQz7EFTK1wJPq/7AwudHNEfU/AeAAAAABJRU5ErkJggg==) no-repeat 99% 52% !important;
	background-size: 10px !important;
}
.container{
  height: 100%;
}
.z1001{
	z-index: 1001;
}
.wrapper {
    position: relative;
    margin: 0 15px 10px 15px;
    text-align: center;
    border-radius: 5px;
    background: #fff;
    border: 1px solid #E6E4E1;
}

.wrapper>div {
    height: 45px;
    line-height: 45px;
    margin: 0 16px;
    font-size: 16px;
    text-align: left;
    color: #999;
    border-bottom: 1px solid #F5F5F5;
}

.wrapper label {
    width: 10em;
    font-family: PingFang-SC-Medium;
    font-size: 15px;
    font-weight: 400;
    color: #333;
}

label {
    position: absolute;
    width: 7em;
	-webkit-tap-highlight-color: transparent; 

}

input {
    width: 100%;
    padding-left: 4em;
    text-align: right;
    box-sizing: border-box;
    border: none;
    outline: none;
	color: #333;
	letter-spacing:0.5px;
	font-size: 15px;
	font-family: PingFang-SC-Medium;
	font-weight: 400;
	-webkit-tap-highlight-color: transparent; 

}
.left {
    float: left;
}
.right{
	float: right;
}
input::placeholder {
    text-align: right;
    color: #ccc;
}

.arrow {
    padding-right: 15px;
    background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAB+klEQVRIS62Wv4sTQRTHv28DdoJ98ldok8ybNHdcd4o22vnjCkUOEeWw9AeKIocgHlYqFuIV8UepnRbZmVVBG0mllZWiRVJ4guB+5UmuS7KzZ6Yadt9+Pvv2vZlZwZQRQrhKcst7f2NaTMp1mRTU7/f3ZVn2FkADwFlVvZMCmxQzUWCBMcYVkvdtTvKk9/7fvO6YKjBQCOEMAHv7PyJy1Dm3OVfBWHIBwE2TkDzsvX9eRzIzg21QjPEKyYsAfpM86L1/mSpJEowzuQXgPIAtksve+9cpkmQBSYkx3gVwWkR+klxS1aJKkiwYd5MURfGQ5DEAoyzLFjqdzodZkloCA/V6vUar1dq0ggP4LiKLzrmP0yS1BQYaDAa7RqPRU5L7SX7NsqzrnPtca6FVfds8z3eLyBcAewC8UdXO3ASWwXA4fALgAIBvjUaj2263P81FYDVoNpuPARwB8ENEFuZWg3GrPgBwwrqoLMvFbrf7fi5dZPAQwoaIrNpiE5El51ysqlVyF+V5vi4iawB+AVhW1VdVcLufJAghXAZwabwXHfLev0iBJwlijGsk1203tcKq6rNUeKUgxrhKcsPOHADHVfVRHfhMQQjBOsU6xsYpVb1XFz5VUBTF3rIs39mZLCLnnHO3dwKfmUGM8br9VajqtZ3CK2vwP+DtZ/8CVbfLGdhcLzIAAAAASUVORK5CYII=) no-repeat 100% 50%;
    background-size: 10px;
	-webkit-tap-highlight-color: transparent; 
}
.ip-disable {
    pointer-events: none;
}

.onOff {
    position: absolute;
    right: 19px;
    top: 13px;
    height: 22px;
    width: 45px;
    background: #fff;
    border-radius: 11px;
    cursor: pointer;
    display: block;
	border: 1px solid rgba(0,0,0,0.1);
}

.onOff_off {
    position: absolute;
    top: 1px;
    left: 1px;
    width: 20px;
    height: 20px;
    background: #fff;
    border-radius: 50%;
	box-shadow: 3px 3px 10px rgba(0,0,0,0.1);
}

.open {
    background: #FF8D1D;
}

.none {
    display: none;
}

.onOff_off.active {
    left: 54%;
}

.calculateBtn {
    width: 50%;
    height: 45px;
    line-height: 45px;
    margin: 20px auto;
    font-size: 17px;
    color: #fff;
    font-weight: 400;
    text-align: center;
    border-radius: 45px;
    background: -webkit-linear-gradient(left, #FF731F, #FFA53A);
    /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(right, #FF731F, #FFA53A);
    /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(right, #FF731F, #FFA53A);
    /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, #FF731F, #FFA53A);
    /* 标准的语法 */
}

.tips {
    position: relative;
    bottom: 0;
    width: 100%;
    height: 40px;
    line-height: 40px;
    font-size: 12px;
    color: #ACA9A7;
    text-align: center;
}

.mb21 {
    margin-bottom: 21px;
}

.right {
    float: right;
}

li {
    list-style: none;
}

.mt20 {
    margin-top: 20px;
}

.left {
    float: left;
}
.cfont{
    color:#FF8D1D;
}
.c-disable{
    color:#ACA9A7;
}

</style>
