# wxRequest

// 加载配置文件
const config = require('config.js');
var app=getApp();
module.exports = {
  /**
   * get方式请求,ulr是请求api号，token是登陆token,不用token就传空，fn是函数成功的回调函数，data为向后台传递的参数by:张涛20180303
   */
  GET: function (url = '',token='' ,data = {}, fn,fail) {
    wx.request({
      url: config.API_HOST + url,//请求地址
      method: 'get',//请求方式
      data: data,//请求参数
      header: { "Content-Type": "application/json" ,'token':token},
      success: function (res) {
        // 判断token是否失效
        if (res.data.code=='JWT00002'||res.data.code=='JWT00001'||res.data.code=='JWT00004'||res.data.code=='403') {
          wx.navigateTo({
              url:'/pages/login/login'
          })
          return false;
        }
        fn(res);
      },
      fail: function (res) {
        // wx.showToast({
        //   title: '请求服务器失败，请稍后再试！',
        //   icon: 'loading',
        //   duration: 2000
        // })
      }
    });
  },

  /**
   * post方式请求
   */
  POST: function (url = '',token='', data = {}, fn ,fail) {
    wx.request({
      url: config.API_HOST + url,//请求地址
      method: 'post',//请求方式
      data: data,//请求参数
      header: { "Content-Type": "application/json",'token':token},
      success: function (res) {
        // 判断token是否失效 如果失效就跳转登录页面
        if (res.data.code=='JWT00002'||res.data.code=='JWT00001'||res.data.code=='JWT00004'||res.data.code=='403') {
          wx.navigateTo({
              url:'/pages/login/login'
          })
          return false;
        }
        fn(res);
      },
      fail: function (res) {
        // wx.showToast({
        //   title: '请求服务器失败，请稍后再试！',
        //   icon: 'loading',
        //   duration: 2000
        // })
      }
    });
  },
  /*
  *时间戳格式修改公共函数
  *timestamp为后台传递的时间戳
  *type为时间显示的不同方式
  *bol：判断是否需要时分秒默认不要
  *主要用来分割年月日
  *后期可以扩展年月日时分秒。
  *by:张涛 20180305
*/

  setTime:function(timestamp,type,bol){
    var unixTimestamp = new Date(timestamp) ;
    // 首先判断是否需要时分秒
    if (bol) {
      //设置不同的格式 
      Date.prototype.toLocaleString = function() {
        return this.getFullYear() + type + (this.getMonth() + 1) + type + this.getDate()+' '+ this.getHours() + ":" + this.getMinutes();
      };
    }else{
      //设置不同的格式 
      Date.prototype.toLocaleString = function() {
        return this.getFullYear() + type + (this.getMonth() + 1) + type + this.getDate();
      };
    }   
    return unixTimestamp.toLocaleString();
  },
  // 时间戳倒计时函数,根据时间戳差值计算剩余时间
  /*
  *时间：timestamp(非毫秒级)，fn回调函数，参数可定义
  *暂时为天小时分钟秒，后期可拓展by:张涛20180305
  *
  *第一种只进行倒计时解析
  *第二种倒计时实时显示
  */
  downTime:function(timestamp,type,fn){
    // 只解析剩余时间
    if (type==1) {
      var time={
        day:'',
        hour:'',
        minute:'',
        second:''
      } 
      time.day=Math.floor(timestamp / (24*3600));
      time.hour=Math.floor((timestamp-time.day*24*3600)/3600);
      time.minute=Math.floor((timestamp-time.day*24*3600-time.hour*3600)/60);
      time.second=Math.floor(timestamp-time.day*24*3600-time.hour*3600-time.minute*60);
      return time;
    }else if (type==2) {
      var day,hour,minute,second,time;
      // 解析剩余时间，并进行动态显示
      var timer = setInterval(function () {
          timestamp--;
          if (time == 0) {
            clearInterval(timer)
          }else{
            day=Math.floor(timestamp / (24*3600));
            hour=Math.floor((timestamp-day*24*3600)/3600);
            minute=Math.floor((timestamp-day*24*3600-hour*3600)/60);
            second=Math.floor(timestamp-day*24*3600-hour*3600-minute*60);
          }
          time={
            day:day,
            hour:hour,
            minute:minute,
            second:second
          }
          //倒计时的回调函数(参数)天，时，分，秒
          fn(time);
        }, 1000)
    }   
  },
  /*
  *检测用户是否登录的函数
  *
  */
  checkLogin:function(){
    if (app.globalData.loginInfo==''||app.globalData.loginInfo=='underfind'||app.globalData.loginInfo==null) {
        wx.navigateTo({
        url:'/pages/login/login'
    })
    // 阻止页面逻辑继续执行
    return false;
    }
    return true;     
  },
  // 检测手机号是否正确
  checkPhone:function(phone){
    var phoneReg=/^1[3|4|5|6|7|8|9][0-9]{9}$/;
    return phoneReg.test(phone);
  },
  // 检测密码验证码中是否存在汉字
  checkCode:function(code){
    var codeReg=/[\u4E00-\u9FA5]/i;
    return codeReg.test(code);
  },
  // 增加页面提示框
  remind:function(title){
    wx.showToast({
        title:title,
        icon: 'none',
        duration: 2000
    })
  },
  // 检测身份证号是否正确
  checkId:function(number){
    var codeId=/(^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$)|(^[1-9]\d{5}\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{2}[0-9Xx]$)/;
    return codeId.test(number)
  }
}

