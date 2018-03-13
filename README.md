# wxRequest


#常用微信小程序公共函数请求函数，验证函数，提醒函数，日期转换


// 加载配置文件
const config = require('config.js');<br>
var app=getApp();<br>
module.exports = {<br>
  /**
   * get方式请求,ulr是请求api号，token是登陆token,不用token就传空，fn是函数成功的回调函数，data为向后台传递的参数by:张涛20180303
   */<br>
  GET: function (url = '',token='' ,data = {}, fn,fail) {<br>
    wx.request({<br>
      url: config.API_HOST + url,//请求地址<br>
      method: 'get',//请求方式<br>
      data: data,//请求参数<br>
      header: { "Content-Type": "application/json" ,'token':token},<br>
      success: function (res) {<br>
        // 判断token是否失效
        if (res.data.code=='JWT00002'||res.data.code=='JWT00001'||res.data.code=='JWT00004'||res.data.code=='403') {<br>
          wx.navigateTo({<br>
              url:'/pages/login/login'<br>
          })<br>
          return false;<br>
        }<br>
        fn(res);<br>
      },<br>
      fail: function (res) {<br>
        // wx.showToast({<br>
        //   title: '请求服务器失败，请稍后再试！',<br>
        //   icon: 'loading',<br>
        //   duration: 2000<br>
        // })<br>
      }<br>
    });<br>
  },<br>

  /**
   * post方式请求
   */
  POST: function (url = '',token='', data = {}, fn ,fail) {<br>
    wx.request({<br>
      url: config.API_HOST + url,//请求地址<br>
      method: 'post',//请求方式<br>
      data: data,//请求参数<br>
      header: { "Content-Type": "application/json",'token':token},<br>
      success: function (res) {<br>
        // 判断token是否失效 如果失效就跳转登录页面<br>
        if (res.data.code=='JWT00002'||res.data.code=='JWT00001'||res.data.code=='JWT00004'||res.data.code=='403') {<br>
          wx.navigateTo({<br>
              url:'/pages/login/login'<br>
          })<br>
          return false;<br>
        }<br>
        fn(res);<br>
      },<br>
      fail: function (res) {<br>
        // wx.showToast({<br>
        //   title: '请求服务器失败，请稍后再试！',<br>
        //   icon: 'loading',<br>
        //   duration: 2000<br>
        // })<br>
      }<br>
    });<br>
  },<br>
  /*
  *时间戳格式修改公共函数
  *timestamp为后台传递的时间戳
  *type为时间显示的不同方式
  *bol：判断是否需要时分秒默认不要
  *主要用来分割年月日
  *后期可以扩展年月日时分秒。
  *by:张涛 20180305
*/

  setTime:function(timestamp,type,bol){<br>
    var unixTimestamp = new Date(timestamp) ;<br>
    // 首先判断是否需要时分秒<br>
    if (bol) {<br>
      //设置不同的格式 <br>
      Date.prototype.toLocaleString = function() {<br>
        return this.getFullYear() + type + (this.getMonth() + 1) + type + this.getDate()+' '+ this.getHours() + ":" + this.getMinutes();<br>
      };<br>
    }else{<br>
      //设置不同的格式 <br>
      Date.prototype.toLocaleString = function() {<br>
        return this.getFullYear() + type + (this.getMonth() + 1) + type + this.getDate();<br>
      };<br>
    }   <br>
    return unixTimestamp.toLocaleString();<br>
  },<br>
  // 时间戳倒计时函数,根据时间戳差值计算剩余时间<br>
  /*
  *时间：timestamp(非毫秒级)，fn回调函数，参数可定义
  *暂时为天小时分钟秒，后期可拓展by:张涛20180305
  *
  *第一种只进行倒计时解析
  *第二种倒计时实时显示
  */
  downTime:function(timestamp,type,fn){<br>
    // 只解析剩余时间<br>
    if (type==1) {<br>
      var time={<br>
        day:'',<br>
        hour:'',<br>
        minute:'',<br>
        second:''<br>
      } <br>
      time.day=Math.floor(timestamp / (24*3600));<br>
      time.hour=Math.floor((timestamp-time.day*24*3600)/3600);<br>
      time.minute=Math.floor((timestamp-time.day*24*3600-time.hour*3600)/60);<br>
      time.second=Math.floor(timestamp-time.day*24*3600-time.hour*3600-time.minute*60);<br>
      return time;<br>
    }else if (type==2) {<br>
      var day,hour,minute,second,time;<br>
      // 解析剩余时间，并进行动态显示<br>
      var timer = setInterval(function () {<br>
          timestamp--;<br>
          if (time == 0) {<br>
            clearInterval(timer)<br>
          }else{<br>
            day=Math.floor(timestamp / (24*3600));<br>
            hour=Math.floor((timestamp-day*24*3600)/3600);<br>
            minute=Math.floor((timestamp-day*24*3600-hour*3600)/60);<br>
            second=Math.floor(timestamp-day*24*3600-hour*3600-minute*60);<br>
          }<br>
          time={<br>
            day:day,<br>
            hour:hour,<br>
            minute:minute,<br>
            second:second<br>
          }<br>
          //倒计时的回调函数(参数)天，时，分，秒<br>
          fn(time);<br>
        }, 1000)<br>
    }   <br>
  },<br>
  /*
  *检测用户是否登录的函数
  *
  */
  checkLogin:function(){<br>
    if (app.globalData.loginInfo==''||app.globalData.loginInfo=='underfind'||app.globalData.loginInfo==null) {<br>
        wx.navigateTo({<br>
        url:'/pages/login/login'<br>
    })<br>
    // 阻止页面逻辑继续执行<br>
    return false;<br>
    }<br>
    return true;     <br>
  },<br>
  // 检测手机号是否正确<br>
  checkPhone:function(phone){<br>
    var phoneReg=/^1[3|4|5|6|7|8|9][0-9]{9}$/;<br>
    return phoneReg.test(phone);<br>
  },<br>
  // 检测密码验证码中是否存在汉字<br>
  checkCode:function(code){<br>
    var codeReg=/[\u4E00-\u9FA5]/i;<br>
    return codeReg.test(code);<br>
  },<br>
  // 增加页面提示框<br>
  remind:function(title){<br>
    wx.showToast({<br>
        title:title,<br>
        icon: 'none',<br>
        duration: 2000<br>
    })<br>
  },<br>
  // 检测身份证号是否正确<br>
  checkId:function(number){<br>
    var codeId=/(^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$)|(^[1-9]\d{5}\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{2}[0-9Xx]$)/;<br>
    return codeId.test(number)<br>
  }<br>
}<br>
<br>

