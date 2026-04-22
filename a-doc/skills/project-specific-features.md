# 项目特定功能开发指南

## 项目概述

**项目名称**: station-admin-miniapp (智慧社区管理小程序)  
**技术栈**: Taro 4.1.11 + NutUI 3.0.18 + React + TypeScript  
**主要功能**: 智慧社区管理、即时通讯、资讯中心、志愿者服务

## 核心功能模块

### 1. OpenIM 即时通讯集成

#### SDK 初始化配置
```typescript
// src/utils/openim.ts
import OpenIM from '@openim/client-sdk';

const defaultIMConfig = {
  wsAddr: 'wss://your-im-server:10001',
  apiAddr: 'https://your-im-server:10002',
};

// 在 app.ts 中初始化
openIMService.init(imConfig);
```

#### IM 连接状态监听
```typescript
import openIMService, { ConnectStatus } from '@/utils/openim';

// 监听连接状态
openIMService.onStatusChange((status) => {
  if (status === ConnectStatus.Connected) {
    console.log('IM 已连接');
  } else if (status === ConnectStatus.Failed) {
    console.error('IM 连接失败');
  }
});

// 清理监听
openIMService.offStatusChange(() => {});
```

#### 消息处理
```typescript
// 监听新消息
openIMService.onMessage((messages) => {
  console.log('收到新消息:', messages);
  // 处理消息通知
});

// 监听自定义业务消息
openIMService.onCustomBusiness((message) => {
  console.log('收到自定义业务消息:', message);
});
```

### 2. 认证授权系统

#### 自动登录 Hook
```typescript
// hooks/useAutoLogin.ts
import useAutoLogin from '@/hooks/useAutoLogin';

function Page() {
  const { checkLogin, isLoggedIn, loginData } = useAutoLogin();
  
  useEffect(() => {
    checkLogin().then(result => {
      if (result) {
        authState.setLoggedIn();
      } else {
        authState.setGuest();
      }
    });
  }, []);
  
  return <View>{isLoggedIn ? '已登录' : '未登录'}</View>;
}
```

#### 登录状态守卫
```typescript
// hooks/useAuthGuard.ts
import { useAuthGuard } from '@/hooks/useAuthGuard';

function ProtectedPage() {
  useAuthGuard(); // 未登录自动跳转
  
  return <View>需要登录的页面内容</View>;
}
```

#### 认证状态管理
```typescript
// utils/auth-state.ts
import authState from '@/utils/auth-state';

// 设置登录状态
authState.setLoggedIn();

// 设置为访客
authState.setGuest();

// 监听状态变化
authState.onAuthChange((state: AuthState) => {
  switch (state) {
    case 'loggedIn':
      // 已登录逻辑
      break;
    case 'guest':
      // 访客逻辑
      break;
    case 'error':
      // 错误状态
      break;
  }
});

// 开始检查循环
authState.startChecking();

// 清理监听器
authState.clearListeners();
```

### 3. 全局消息通知组件

#### IM 消息通知
```typescript
// components/IMMessageNotification/index.tsx
import IMMessageNotification from '@/components/IMMessageNotification';

function App() {
  return (
    <React.Fragment>
      {children}
      <IMMessageNotification />
    </React.Fragment>
  );
}
```

#### 自定义业务通知
```typescript
// components/CustomBusinessNotification/index.tsx
import CustomBusinessNotification from '@/components/CustomBusinessNotification';

// 处理自定义业务消息通知
<CustomBusinessNotification 
  onReceive={(message) => {
    // 处理业务消息
  }}
/>
```

#### 求助任务弹窗
```typescript
// components/HelpTaskModal/index.tsx
import HelpTaskModal from '@/components/HelpTaskModal';

function App() {
  return (
    <React.Fragment>
      {children}
      <HelpTaskModal />
    </React.Fragment>
  );
}
```

### 4. 页面功能示例

#### 首页（Index）
```typescript
// pages/index/index.tsx
import Taro from '@tarojs/taro';
import { View, Text } from '@tarojs/components';
import { Button, Cell } from '@nutui/nutui-react-taro';
import { useAuthGuard } from '@/hooks/useAuthGuard';

export default function Index() {
  useAuthGuard(); // 登录守卫
  
  const handleNavigate = (url: string) => {
    Taro.navigateTo({ url });
  };
  
  return (
    <View className="index-page">
      <Cell title="资讯中心" isLink onClick={() => handleNavigate('/pages/information/index')} />
      <Cell title="志愿者服务" isLink onClick={() => handleNavigate('/pages/conversation/volunteer-help')} />
      <Cell title="个人中心" isLink onClick={() => handleNavigate('/pages/mine/index')} />
      
      <Button type="primary" block onClick={handleLogout}>
        退出登录
      </Button>
    </View>
  );
}
```

#### 资讯中心页面
```typescript
// pages/information/index.tsx
import { useState, useEffect } from 'react';
import Taro from '@tarojs/taro';
import { ScrollView } from '@tarojs/components';
import { Cell, Skeleton } from '@nutui/nutui-react-taro';
import { getInformationList } from '@/services/information';

interface InformationItem {
  id: string;
  title: string;
  publishTime: string;
  category: 'news' | 'knowledge';
}

export default function Information() {
  const [loading, setLoading] = useState(true);
  const [list, setList] = useState<InformationItem[]>([]);
  
  useEffect(() => {
    loadInformation();
  }, []);
  
  const loadInformation = async () => {
    try {
      const res = await getInformationList();
      setList(res.data);
    } catch (error) {
      Taro.showToast({ title: '加载失败', icon: 'error' });
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <ScrollView scrollY className="information-page">
      {loading ? (
        <Skeleton rows={10} />
      ) : (
        list.map(item => (
          <Cell
            key={item.id}
            title={item.title}
            description={item.publishTime}
            isLink
            onClick={() => {
              Taro.navigateTo({
                url: `/pages/information/${item.category === 'news' ? 'news_info' : 'knowledge_info'}?id=${item.id}`
              });
            }}
          />
        ))
      )}
    </ScrollView>
  );
}
```

#### 志愿者帮助页面
```typescript
// pages/conversation/volunteer-help.tsx
import { useState } from 'react';
import Taro from '@tarojs/taro';
import { Form, Input, Button, TextArea, Picker } from '@nutui/nutui-react-taro';

export default function VolunteerHelp() {
  const [formData, setFormData] = useState({
    type: '',
    content: '',
    contact: '',
  });
  
  const handleSubmit = async () => {
    try {
      // 调用 API 提交求助
      await Taro.request({
        url: `${API_GATEWAY}/volunteer/help`,
        method: 'POST',
        data: formData,
      });
      
      Taro.showToast({ title: '提交成功', icon: 'success' });
      Taro.navigateBack();
    } catch (error) {
      Taro.showToast({ title: '提交失败', icon: 'error' });
    }
  };
  
  return (
    <Form className="volunteer-help-page">
      <Form.Item label="求助类型" name="type">
        <Picker
          columns={['生活帮助', '医疗咨询', '心理疏导', '其他']}
          onConfirm={(value) => setFormData({ ...formData, type: value })}
        >
          <Input placeholder="请选择求助类型" />
        </Picker>
      </Form.Item>
      
      <Form.Item label="求助内容" name="content">
        <TextArea
          placeholder="请详细描述您的需求"
          maxLength={500}
          showCount
          onChange={(value) => setFormData({ ...formData, content: value })}
        />
      </Form.Item>
      
      <Form.Item label="联系方式" name="contact">
        <Input
          placeholder="手机号或微信号"
          type="text"
          onChange={(value) => setFormData({ ...formData, contact: value })}
        />
      </Form.Item>
      
      <Button type="primary" block onClick={handleSubmit}>
        提交求助
      </Button>
    </Form>
  );
}
```

#### 个人中心页面
```typescript
// pages/mine/index.tsx
import { useEffect, useState } from 'react';
import Taro from '@tarojs/taro';
import { View } from '@tarojs/components';
import { Cell, Avatar } from '@nutui/nutui-react-taro';
import authState from '@/utils/auth-state';

export default function Mine() {
  const [userInfo, setUserInfo] = useState(null);
  
  useEffect(() => {
    loadUserInfo();
  }, []);
  
  const loadUserInfo = async () => {
    const storage = await Taro.getStorage({ key: 'userInfo' });
    setUserInfo(storage.data);
  };
  
  const handleLogout = () => {
    Taro.showModal({
      title: '提示',
      content: '确定要退出登录吗？',
      success: async (res) => {
        if (res.confirm) {
          await Taro.removeStorage({ key: 'userInfo' });
          authState.setGuest();
          Taro.redirectTo({ url: '/pages/login/index' });
        }
      },
    });
  };
  
  return (
    <View className="mine-page">
      <View className="user-header">
        <Avatar src={userInfo?.avatar} size="large" />
        <View className="user-info">
          <Text className="nickname">{userInfo?.nickname}</Text>
          <Text className="phone">{userInfo?.phone}</Text>
        </View>
      </View>
      
      <Cell.Group title="我的服务">
        <Cell title="健康档案" isLink onClick={() => Taro.navigateTo({ url: '/pages/mine/health-archive' })} />
        <Cell title="志愿记录" isLink onClick={() => Taro.navigateTo({ url: '/pages/mine/record' })} />
        <Cell title="账号信息" isLink onClick={() => Taro.navigateTo({ url: '/pages/mine/account/info' })} />
      </Cell.Group>
      
      <Cell.Group title="设置">
        <Cell title="志愿者认证" isLink onClick={() => Taro.navigateTo({ url: '/pages/mine/volunteer/authentication' })} />
        <Cell title="呼叫通知设置" isLink onClick={() => Taro.navigateTo({ url: '/pages/mine/volunteer/callNoticeSetting' })} />
        <Cell title="设置" isLink onClick={() => Taro.navigateTo({ url: '/pages/mine/setting/index' })} />
      </Cell.Group>
      
      <Button type="danger" block onClick={handleLogout}>
        退出登录
      </Button>
    </View>
  );
}
```

### 5. API 服务层

#### 认证服务
```typescript
// services/auth.ts
import Taro from '@tarojs/taro';

interface LoginParams {
  code: string;
  userInfo?: {
    nickName: string;
    avatarUrl: string;
  };
}

interface LoginResponse {
  token: string;
  userInfo: {
    userId: string;
    nickname: string;
    avatar: string;
    phone?: string;
  };
}

export const authApi = {
  // 微信登录
  login: (params: LoginParams): Promise<LoginResponse> => {
    return Taro.request({
      url: `${API_GATEWAY}/auth/login`,
      method: 'POST',
      data: params,
    }).then(res => res.data);
  },
  
  // 刷新令牌
  refreshToken: (): Promise<LoginResponse> => {
    return Taro.request({
      url: `${API_GATEWAY}/auth/refresh`,
      method: 'POST',
    }).then(res => res.data);
  },
  
  // 获取用户信息
  getUserInfo: (): Promise<LoginResponse['userInfo']> => {
    return Taro.request({
      url: `${API_GATEWAY}/user/info`,
      method: 'GET',
    }).then(res => res.data);
  },
};
```

#### 自动登录服务
```typescript
// services/auto-login.ts
import Taro from '@tarojs/taro';
import { authApi } from './auth';

export const autoLoginService = {
  // 检查本地缓存
  checkLocalAuth: async (): Promise<boolean> => {
    try {
      const storage = await Taro.getStorage({ key: 'userInfo' });
      return !!storage.data?.token;
    } catch {
      return false;
    }
  },
  
  // 执行自动登录
  performAutoLogin: async (): Promise<boolean> => {
    try {
      const result = await authApi.refreshToken();
      if (result.token) {
        await Taro.setStorage({
          key: 'userInfo',
          data: result,
        });
        return true;
      }
      return false;
    } catch {
      return false;
    }
  },
  
  // 微信授权登录
  wechatAuthorize: async (): Promise<boolean> => {
    try {
      // 获取微信登录码
      const { code } = await Taro.wxLogin();
      
      // 获取用户信息
      const userProfile = await new Promise<any>((resolve) => {
        Taro.getUserProfile({
          desc: '用于完善用户资料',
          success: resolve,
          fail: () => resolve({}),
        });
      });
      
      // 调用后端登录
      const result = await authApi.login({
        code,
        userInfo: {
          nickName: userProfile.nickName,
          avatarUrl: userProfile.avatarUrl,
        },
      });
      
      if (result.token) {
        await Taro.setStorage({
          key: 'userInfo',
          data: result,
        });
        return true;
      }
      
      return false;
    } catch {
      return false;
    }
  },
};
```

#### 信息服务
```typescript
// services/information.ts
import Taro from '@tarojs/taro';

interface InformationItem {
  id: string;
  title: string;
  content: string;
  publishTime: string;
  category: 'news' | 'knowledge';
}

export const informationApi = {
  // 获取资讯列表
  getList: (category?: 'news' | 'knowledge'): Promise<InformationItem[]> => {
    return Taro.request({
      url: `${API_GATEWAY}/information/list`,
      method: 'GET',
      data: { category },
    }).then(res => res.data);
  },
  
  // 获取资讯详情
  getDetail: (id: string): Promise<InformationItem> => {
    return Taro.request({
      url: `${API_GATEWAY}/information/detail`,
      method: 'GET',
      data: { id },
    }).then(res => res.data);
  },
};
```

### 6. 工具函数

#### 请求封装
```typescript
// utils/request.ts
import Taro from '@tarojs/taro';
import authState from './auth-state';

interface RequestOptions {
  url: string;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  data?: any;
  header?: any;
  needAuth?: boolean;
}

export const request = async <T>(options: RequestOptions): Promise<T> => {
  const { url, method = 'GET', data, header = {}, needAuth = true } = options;
  
  // 获取 token
  let token = '';
  if (needAuth) {
    try {
      const storage = await Taro.getStorage({ key: 'userInfo' });
      token = storage.data?.token || '';
    } catch {
      // 未登录，跳转到登录页
      authState.setGuest();
      Taro.redirectTo({ url: '/pages/login/index' });
      throw new Error('未登录');
    }
  }
  
  // 显示加载提示
  Taro.showLoading({ title: '加载中...' });
  
  try {
    const res = await Taro.request({
      url,
      method,
      data,
      header: {
        'Content-Type': 'application/json',
        ...(token ? { 'Authorization': `Bearer ${token}` } : {}),
        ...header,
      },
    });
    
    // 检查响应码
    if (res.data.code !== 200) {
      Taro.showToast({
        title: res.data.message || '请求失败',
        icon: 'error',
      });
      
      // 401 未登录
      if (res.data.code === 401) {
        authState.setGuest();
        Taro.redirectTo({ url: '/pages/login/index' });
      }
      
      throw new Error(res.data.message);
    }
    
    return res.data.data;
  } catch (error) {
    console.error('请求错误:', error);
    throw error;
  } finally {
    Taro.hideLoading();
  }
};
```

#### 地图工具
```typescript
// utils/mapUtils.ts
import Taro from '@tarojs/taro';

export const mapUtils = {
  // 获取当前位置
  getCurrentLocation: async (): Promise<{ latitude: number; longitude: number }> => {
    return new Promise((resolve, reject) => {
      Taro.getLocation({
        type: 'gcj02',
        success: resolve,
        fail: reject,
      });
    });
  },
  
  // 打开地图选择位置
  chooseLocation: async (): Promise<any> => {
    return new Promise((resolve, reject) => {
      Taro.chooseLocation({
        success: resolve,
        fail: reject,
      });
    });
  },
  
  // 打开内置地图
  openLocation: async (location: { latitude: number; longitude: number; name?: string; address?: string }) => {
    await Taro.openLocation({
      ...location,
      scale: 18,
    });
  },
  
  // 使用高德地图 API
  getAmapLocation: async (address: string) => {
    // 使用 amap-wx.130.js
    const myAmap = new AMapWX.AMapWeather({ key: 'your-amap-key' });
    return myAmp.getWeather({ city: address });
  },
};
```

### 7. 配置文件

#### 页面配置
```typescript
// pages/index/index.config.ts
export default definePageConfig({
  navigationBarTitleText: '首页',
  enablePullDownRefresh: true,
  backgroundTextStyle: 'dark',
});
```

#### 应用配置
```typescript
// app.config.ts
export default {
  pages: [
    'pages/index/index',
    'pages/information/index',
    'pages/information/news_info',
    'pages/information/knowledge_info',
    'pages/conversation/volunteer-help',
    'pages/mine/index',
    'pages/mine/health-archive',
    'pages/mine/record',
    'pages/mine/account/info',
    'pages/mine/volunteer/authentication',
    'pages/mine/volunteer/callNoticeSetting',
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: '智慧社区',
    navigationBarTextStyle: 'black',
  },
  tabBar: {
    color: '#999',
    selectedColor: '#fa541c',
    backgroundColor: '#fff',
    list: [
      {
        pagePath: 'pages/index/index',
        text: '首页',
        iconPath: 'src/assets/img/tabbar/index.png',
        selectedIconPath: 'src/assets/img/tabbar/index_active.png',
      },
      {
        pagePath: 'pages/information/index',
        text: '资讯',
        iconPath: 'src/assets/img/tabbar/news.png',
        selectedIconPath: 'src/assets/img/tabbar/news_active.png',
      },
      {
        pagePath: 'pages/mine/index',
        text: '我的',
        iconPath: 'src/assets/img/tabbar/mine.png',
        selectedIconPath: 'src/assets/img/tabbar/mine_active.png',
      },
    ],
  },
};
```

## 开发注意事项

### 1. NutUI 主题定制
- 项目使用 JMAPP 主题 (`variables-jmapp.scss`)
- 修改主题变量需编辑对应的 SCSS 文件
- 确保 `designWidth` 配置正确（NutUI 组件使用 375）

### 2. OpenIM 消息推送
- 在 `app.ts` 中全局监听 IM 消息
- 使用通知组件统一处理消息展示
- 注意处理 IM 连接状态和重连逻辑

### 3. 登录状态管理
- 使用 `auth-state` 统一管理登录状态
- 自动登录失败时降级为访客模式
- 需要登录的页面使用 `useAuthGuard` Hook

### 4. 性能优化
- 列表页使用虚拟滚动
- 图片使用懒加载
- 避免在渲染函数中创建新对象

### 5. 微信小程序限制
- 注意包大小限制（主包 2MB）
- 合理使用分包加载
- 遵循微信小程序审核规范

## 测试与调试

### 开发环境
```bash
# 启动微信小程序开发模式
npm run dev:weapp

# 构建生产版本
npm run build:weapp
```

### 调试技巧
1. 使用微信开发者工具的调试模式
2. 开启 SourceMap 便于调试
3. 使用 `console.log` 输出关键信息
4. 真机预览测试兼容性

## 部署流程

1. 开发完成后运行 `npm run build:weapp`
2. 在微信开发者工具中预览
3. 上传代码到微信小程序后台
4. 提交审核
5. 发布上线

## 参考资源

- 项目已有的组件和 Hooks
- Taro 官方文档
- NutUI 组件库文档
- OpenIM SDK 文档
- 微信小程序开发文档
