## 接口相关

#### 太平接口文档地址 ：http://wxtest.life.cntaiping.com/tpbb/swagger-ui.html#/user-controller/getUserInfoUsingGET

#### 获取代理人信息接口

​     /tpbb/user/user-data 

这个接口会获取用户信息, 他分两种情况,一种是代理人信息,一种是微信信息

这个接口会判断域名下的 cookie ,如果是代理人登录的 cookie 就会返回代理人信息, 如果是微信认证的 cookie 就会重定向到另一个接口返回微信的信息.

#### 机构代码 机构名称 

101 上海分公司 
102 北京分公司 
103 广东分公司 
104 四川分公司 
105 河北分公司 
106 河南分公司 
107 江苏分公司 
108 山东分公司 
109 浙江分公司 
110 辽宁分公司 
111 宁波分公司 
112 深圳分公司 
113 青岛分公司 
114 大连分公司 
115 佛山分公司 
116 苏州分公司 
117 天津分公司 
118 湖北分公司 
119 安徽分公司 
120 福建分公司 
121 黑龙江分公司 
122 江西分公司 
123 厦门分公司(归福建) 
124 重庆分公司 
125 湖南分公司 
126 陕西分公司 
127 山西分公司 
128 云南分公司 
129 吉林分公司 
130 广西分公司 
131 新疆分公司 
132 贵州分公司 
133 甘肃分公司 
134 内蒙古分公司 
135 海南分公司 
136 青海分公司



## app 提供SDK方法 

下面代码可直接复制使用 getUserInfor 为上接口获取用户信息 

bbAppSDK = 宝保APP方法SDK类

SDKyxxApp = e行销 ｜ 奔驰 方法SDK类 （除SDKyxxApp 内部方法 其他皆为宝保APP的方法）

```javascript
import { getUserInfor } from '@/services/user';

/**
 * 判断当前是否处于 APP 内
 */

export function isInAPP() {
  const userAgent = getUserAgent();

  return userAgent === 'ANDROID_APP' || userAgent === 'IOS_APP';
}

/**
 * 根据太平提供文档，navigation.userAgent 包含 app/2 为 Android 包含 app/3 为 IOS
 * @param ANDROID_APP 安卓app
 * @param IOS_APP ios APP
 * @param WX 微信浏览器
 * @param GAIMARKET 奔驰（易行销）
 */
export const getUserAgent = (): any => {
  const isInUserAgent = (str: string) => {
    return navigator.userAgent.indexOf(str) !== -1;
  };

  if (isInUserAgent('app/2')) return 'ANDROID_APP';

  if (isInUserAgent('app/3')) return 'IOS_APP';

  if (isInUserAgent('MicroMessenger')) return 'WX';
  if (isInUserAgent('gaimarket')) return 'GAIMARKET';
  console.log('当前环境不匹配 微信 和 APP');
  return;
};

interface shareToMiniprogramType {
  webpageUrl?: string;
  path?: string;
  userName?: string;
  title: string;
  desc?: string;
}
/**
 * 从app 分享小程序
 * @param webpageUrl 兼容低版本的网页链接
 * @param path 小程序的 path
 * @param userName 小程序的原始 id
 * @param title 分享标题
 * @param desc 内容描述
 */
export const shareToMiniprogram = async (data?: shareToMiniprogramType) => {
  try {
    /**
     * 小程序 小程序的原始 id
     *  生产：gh_6536682f7121
     *  UAT：gh_f31789840e28
     */
    let user: any = {};
    try {
      user = (await getUserInfor()) || {};
    } catch (error) {
      console.error(error);
    }

    const userAgent = getUserAgent();
    let defaults = {
      title: `${user.realName || ''}邀请您理想保额评测`,
      userName: REACT_APP_ENV === 'dev' ? 'gh_f31789840e28' : 'gh_6536682f7121',
      webpageUrl: 'https://img.life.cntaiping.com/tpbb/scs/pro/review/index.html#/review/start',
      thumb: 'https://img.life.cntaiping.com/tpbb/iFamily/image/index_share.png',
      miniprogramType: REACT_APP_ENV === 'dev' ? 3 : 1,
      path: `/pages/Grantapplication/Grantapplication?type=baobao&&miniFlag=PWC&&agentCode=${user.inviteCode}`,
    };
    if (userAgent == 'ANDROID_APP') {
      //@ts-ignore
      window.tpbbAppRemote.shareToMiniprogram(JSON.stringify(Object.assign(defaults, data || {})));
      return;
    }

    if (userAgent == 'IOS_APP') {
      //@ts-ignore
      window.webkit.messageHandlers.shareToMiniprogram.postMessage(
        JSON.stringify(Object.assign(defaults, data || {})),
      );
      return;
    }
  } catch (error) {
    console.error(error.message);
    alert('分享失败');
  }
};

/**
 * app 内部提供的方法，默认打开一个新页面，也可以传入参数决定是否分享。
 * 用于在app页面新建窗口解决返回直接退出机器人的问题。也可以用来在APP内调用分享
 * @param title - 文件名称
 * @param ext - 文件类型
 * @param url - 下载地址
 * @param shareAction - 是否显示分享图标 0 不允许 1 允许
 * @param shareResource - 分享展示内容
 */
export function APPOpenFile(title: string | null, ext: string, url: string, shareResource?: any) {
  const userAgent = getUserAgent();
  const paramObj = {
    fileTitle: title,
    fileExt: ext,
    fileUrl: url,
    shareAction: shareResource ? 1 : 0,
    shareResource,
  };
  console.log(`ios app 跳转${userAgent}`, JSON.stringify(paramObj));
  // return
  console.log(JSON.stringify(paramObj));
  if (userAgent == 'ANDROID_APP') {
    //@ts-ignore
    return window.tpbbAppRemote.openFile(JSON.stringify(paramObj));
  }

  if (userAgent == 'IOS_APP') {
    if (!paramObj.fileTitle) {
      paramObj.fileTitle = document.title;
    }
    //@ts-ignore
    return window.webkit.messageHandlers.openFile.postMessage(JSON.stringify(paramObj));
  }

  console.error('当前环境不匹配无法进行分享');
}

/**
 * 打开新页面
 * @param title - 页面标题
 * @param path - 页面地址
 */
export const handleAppOpenFile = (path: string, title: string | null) => {
  if (isInAPP()) {
    console.log('app打开页面', path);
    APPOpenFile(title, 'html', path);
  } else {
    console.log('打开新页面', path);
    window.open(path);
  }
};

/**
 * 保宝app 立即投保
 * @param product  太平福禄全能保2.0重大疾病保险 = 45 | 太平福禄双甲终身重大疾病保险 = 50
 */
export const oneKeyInsurance = (product: number | string) => {
  const userAgent = getUserAgent();

  if (userAgent == 'ANDROID_APP') {
    //@ts-ignore
    return window.tpbbAppRemote.oneKeyInsurance(JSON.stringify({ product }));
  }

  if (userAgent == 'IOS_APP') {
    //@ts-ignore
    return window.webkit.messageHandlers.oneKeyInsurance.postMessage(JSON.stringify({ product }));
  }
  return alert('请在保宝app内，进行投保。');
};

/**
 * 宝保app 提供 skd
 */
export class SDKbbApp {
  SDK: any;
  userAgent: any;
  constructor() {
    this.init();
  }
  init() {
    this.userAgent = getUserAgent();
    if (this.userAgent == 'ANDROID_APP') {
      //@ts-ignore
      this.SDK = window.tpbbAppRemote;
      return;
    }
    if (this.userAgent == 'IOS_APP') {
      //@ts-ignore
      this.SDK = window.webkit.messageHandlers;
      return;
    }
  }
/**
 * @function remoteOnShare 分享网页链接 到微信
 * @param params 文档地址 http://wxtest.life.cntaiping.com/tpbb/docs/develop/jsbridge.html#api
 * 无法进入文档 请先点下方进行微商城登录
 * http://wxtest.life.cntaiping.com/tpbb/app-login/login?unionId=ooTTjvqfe7OTGeex18l1Oa0gRrYw
 */
  remoteOnShare(
    params = {
      channel: 'customer-journey',
      shareTo: ['app-message', 'timeline'],
      shareTitle: '享受太平生活，回忆幸福旅程',
      shareInfo: '太平人寿幸福旅历精彩展开，细读您与我们的点滴美好！',
      shareUrl: 'https://life.cntaiping.com',
      iconUrl: 'http://img.life.cntaiping.com/tpbb/app/img/wechat-customer-journey.jpg',
    },
  ) {
 
    if (this.userAgent == 'IOS_APP') {
      this.SDK.remoteOnShare.postMessage(JSON.stringify(params));
    }
    if (this.userAgent == 'ANDROID_APP') {
      this.SDK.remoteOnShare(JSON.stringify(params));
    }
  }
}
export const bbAppSDK = new SDKbbApp();

/**
 * e行销（奔驰）提供 skd
 */
interface shareConfigType {
  scene: '1' | '2' | number; // String 分享类型：1 微信好友，2 朋友圈
  shareType: '0' | '1' | '2' | '3' | '4' | '5'; // String 分享的类型 0-网页url，1-图片，2-video，3-文本，4-music,5-文件
  shareUrl: String; // String 分享链接
  shareTitle: String; // 分享标题
  shareInfo: String; // 分享描述文字
  shareIcon: String; // 分享的图标，支持 url/Base64 （图片大小须小于32KB）
}

export class SDKyxxApp {
  SDK: any;
  plat: any;
  constructor() {
    this.init();
  }
  init() {
    this.plat = this.judgePlatForApp();
    // 调用原生微信分享方法
    if (this.plat === 'ios') {
      //@ts-ignore
      this.SDK = window.webkit.messageHandlers;
    } else if (this.plat === 'android') {
      //@ts-ignore
      this.SDK = window.android;
    }
    console.log('SDK', this.SDK);
  }
  /**
   * share分享
   * @param config 分享配置
   *   scene: '1' | '2' , // String 分享类型：1 微信好友，2 朋友圈
   *   shareType: "0" | '1' | '2' | '3' | '4' | '5', // String 分享的类型 0-网页url，1-图片，2-video，3-文本，4-music,5-文件
   *   shareUrl: String, // String 分享链接
   *   shareTitle: String, // 分享标题
   *   shareInfo: String, // 分享描述文字
   *   shareIcon: String, // 分享的图标，支持 url/Base64 （图片大小须小于32KB）
   */
  public onShare(config: shareConfigType) {
    console.log('分享', config);
    // 调用原生微信分享方法
    if (this.plat === 'ios') {
      //@ts-ignore
      this.SDK.wxShare.postMessage(config);
    } else if (this.plat === 'android') {
      //@ts-ignore
      this.SDK.wxShare(JSON.stringify(config));
    }
  } 
  // 工具函数 判断是否是微信浏览器
  isWx() {
    var ua = navigator.userAgent.toLowerCase();
    //@ts-ignore
    return ua.match(/MicroMessenger/i) == 'micromessenger';
  }
  // 工具函数 判断是否在App内
  isApp() {
    //@ts-ignore
    if ((window.webkit || window.android) && !this.isWx()) return true;
    else return false;
  }
  // 工具函数 判断ios还是android还是web
  judgePlatForApp() {
    var u = navigator.userAgent;
    var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1; //android终端
    var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端

    return !this.isApp() ? 'web' : isiOS ? 'ios' : 'android';
  }
}
export const yxxAppSDK = new SDKyxxApp();

```























   