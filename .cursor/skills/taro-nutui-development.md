# Taro + NutUI 微信小程序开发技能

## 项目技术栈

- **框架**: Taro 4.1.11 (React)
- **UI 组件库**: NutUI 3.0.18 (@nutui/nutui-react-taro)
- **语言**: TypeScript 5.1+
- **样式**: Sass/SCSS
- **目标平台**: 微信小程序

## 开发规范

### 1. 项目结构

```
src/
├── components/          # 公共组件
├── pages/              # 页面目录
├── services/           # API 服务
├── utils/              # 工具函数
├── hooks/              # 自定义 Hooks
├── assets/             # 静态资源
└── config/             # 配置文件
```

### 2. NutUI 组件使用规范

#### 组件导入
```typescript
// ✅ 推荐：按需导入
import { Button, Cell, Dialog } from '@nutui/nutui-react-taro';

// ❌ 不推荐：全量导入
import * as NutUI from '@nutui/nutui-react-taro';
```

#### 常用组件示例

**按钮组件**
```tsx
import { Button } from '@nutui/nutui-react-taro';

<Button type="primary" size="small">主要按钮</Button>
```

**表单组件**
```tsx
import { Form, Input, Button } from '@nutui/nutui-react-taro';

<Form>
  <Form.Item label="用户名" name="username">
    <Input placeholder="请输入用户名" />
  </Form.Item>
</Form>
```

**弹窗组件**
```tsx
import { Dialog } from '@nutui/nutui-react-taro';

Dialog.show({
  title: '提示',
  content: '这是一条提示信息',
  onConfirm: () => console.log('确认'),
});
```

### 3. Taro API 使用规范

#### 页面导航
```typescript
import Taro from '@tarojs/taro';

// 跳转到新页面
Taro.navigateTo({ url: '/pages/detail/index' });

// 返回上一页
Taro.navigateBack();

// 切换 Tab
Taro.switchTab({ url: '/pages/index/index' });
```

#### 网络请求
```typescript
import request from '../utils/request'
import { ApiResponse } from './auth'

/**
 * 场景标签
 */
export interface SceneTag {
  id: string;
  itemText: string;
  itemValue: string;
}

/**
 * 获取场景标签字典
 */
export const getSceneTags = () => {
  return request.get<ApiResponse<SceneTag[]>>('/sys/dict/getDictItemByCode', { 
    dictCode: 'scene_tags' 
  })
}

```

#### 数据存储
```typescript
// 存储数据
await Taro.setStorage({ key: 'userInfo', data: userInfo });

// 获取数据
const userInfo = await Taro.getStorage({ key: 'userInfo' });

// 清除数据
await Taro.removeStorage({ key: 'userInfo' });
```

### 4. React Hooks 使用规范

#### 页面生命周期
```typescript
import { useEffect, useState } from 'react';
import Taro, { useDidShow, useDidHide } from '@tarojs/taro';

function Page() {
  const [data, setData] = useState(null);
  
  // Taro 页面生命周期
  useDidShow(() => {
    console.log('页面显示');
  });
  
  useDidHide(() => {
    console.log('页面隐藏');
  });
  
  // React Hooks
  useEffect(() => {
    // 初始化逻辑
    return () => {
      // 清理逻辑
    };
  }, []);
  
  return <View>页面内容</View>;
}
```

### 5. 样式编写规范

#### SCSS 使用
```scss
// ✅ 推荐：使用 BEM 命名
.container {
  &__header {
    &--active {
      color: red;
    }
  }
  
  &__content {
    padding: 20px;
  }
}

// 使用 NutUI 混入
@import '@nutui/nutui-react-taro/dist/styles/variables-jmapp.scss';

.custom-class {
  @include nut-theme(light);
  background-color: $color-primary;
}
```

#### 响应式适配
```scss
// Taro 自动转换 px 到 rpx
.container {
  width: 750px;  // 实际为 750rpx
  padding: 30px; // 实际为 30rpx
}

// NutUI 组件样式覆盖
:nest(.nut-button) {
  height: 80px;
}
```

### 6. TypeScript 类型定义

#### Props 类型
```typescript
interface IProps {
  title: string;
  onConfirm?: () => void;
  visible: boolean;
}

const MyComponent: React.FC<IProps> = ({ title, onConfirm, visible }) => {
  return <View>{title}</View>;
};
```

#### API 响应类型
```typescript
interface UserInfo {
  userId: string;
  nickname: string;
  avatar: string;
}

interface ApiResponse<T> {
  code: number;
  data: T;
  message: string;
}
```

### 7. 状态管理

#### 使用 auth-state 工具
```typescript
import authState from '@/utils/auth-state';

// 设置登录状态
authState.setLoggedIn();

// 设置为访客
authState.setGuest();

// 监听状态变化
authState.onAuthChange((state) => {
  console.log('认证状态变化:', state);
});
```

#### 自定义 Hooks
```typescript
// hooks/useAuthGuard.ts
import { useEffect } from 'react';
import Taro from '@tarojs/taro';
import authState from '@/utils/auth-state';

export function useAuthGuard() {
  useEffect(() => {
    const unsubscribe = authState.onAuthChange((state) => {
      if (state === 'guest') {
        Taro.redirectTo({ url: '/pages/login/index' });
      }
    });
    
    return () => unsubscribe();
  }, []);
}
```

### 8. 文件命名规范

- **组件文件**: `index.tsx` + `index.scss`
- **页面文件**: `index.tsx` + `index.scss` + `index.config.ts`
- **工具文件**: 小驼峰命名，如 `mapUtils.ts`
- **服务文件**: 小写 + 连字符，如 `auth-service.ts`

### 9. 代码质量检查

#### ESLint 规则
- 使用 TypeScript 类型检查
- React Hooks 规则检查
- Import 顺序规范

#### 提交前检查
```bash
# 运行构建检查
npm run dev:weapp

# 检查 TypeScript 类型
npx tsc --noEmit
```

### 10. 调试技巧

#### 控制台输出
```typescript
console.log('[ComponentName] 调试信息:', data);
console.error('[ComponentName] 错误信息:', error);
console.warn('[ComponentName] 警告信息:', warning);
```

#### 使用 Taro 调试 API
```typescript
Taro.showToast({
  title: '操作成功',
  icon: 'success',
});

Taro.showLoading({ title: '加载中' });
Taro.hideLoading();
```

## 常见问题解决

### 1. NutUI 组件样式不生效
- 检查是否正确导入样式：`@import '@nutui/nutui-react-taro/dist/styles/theme-jmapp.scss';`
- 确保 designWidth 配置正确（NutUI 使用 375）

### 2. Taro API 在真机调试失败
- 检查是否在正确的生命周期调用
- 确保权限配置完整
- 使用微信开发者工具的调试模式

### 3. TypeScript 类型错误
- 使用 `any` 作为临时方案
- 完善接口类型定义
- 使用类型推断减少显式声明

## 最佳实践

1. **组件复用**: 提取通用组件到 `components` 目录
2. **代码分割**: 大型组件拆分为多个子组件
3. **性能优化**: 使用 `React.memo` 避免不必要的渲染
4. **错误处理**: 统一的错误处理和用户提示
5. **可访问性**: 遵循微信小程序无障碍指南

## 开发流程

1. 创建页面/组件
2. 编写 TypeScript 类型定义
3. 实现业务逻辑
4. 添加样式（使用 NutUI 组件优先）
5. 测试功能（开发工具 + 真机）
6. 代码审查
7. 构建发布

## 参考文档

- [Taro 官方文档](https://taro-docs.jd.com/)
- [NutUI 文档](https://nutui.jd.com/#/zh-CN/guide/introduce-taro)
- [微信小程序文档](https://developers.weixin.qq.com/miniprogram/dev/framework/)
