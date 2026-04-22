# Taro + NutUI 组件开发指南

## 组件开发规范

### 1. 组件目录结构

```
src/components/
├── ComponentName/
│   ├── index.tsx      # 组件主体
│   ├── index.scss     # 组件样式
│   └── README.md      # 组件说明（可选）
```

### 2. 组件模板

#### 基础组件模板
```typescript
import React, { FC, useState, useEffect } from 'react';
import { View, Text } from '@tarojs/components';
import { Button } from '@nutui/nutui-react-taro';
import './index.scss';

interface IProps {
  /** 标题 */
  title: string;
  /** 是否可见 */
  visible: boolean;
  /** 确认回调 */
  onConfirm?: () => void;
  /** 取消回调 */
  onCancel?: () => void;
}

/**
 * 组件说明
 * @description 用于 XXX 场景的组件
 * @example
 * <MyComponent 
 *   title="示例标题"
 *   visible={true}
 *   onConfirm={() => console.log('确认')}
 * />
 */
const MyComponent: FC<IProps> = ({
  title,
  visible,
  onConfirm,
  onCancel,
}) => {
  const [localState, setLocalState] = useState(false);
  
  // 生命周期：组件挂载时
  useEffect(() => {
    console.log('组件已挂载');
    return () => {
      console.log('组件卸载');
    };
  }, []);
  
  // 生命周期：visible 变化时
  useEffect(() => {
    if (visible) {
      console.log('组件显示');
    } else {
      console.log('组件隐藏');
    }
  }, [visible]);
  
  const handleClick = () => {
    onConfirm?.();
  };
  
  if (!visible) return null;
  
  return (
    <View className="my-component">
      <View className="my-component__header">
        <Text className="my-component__title">{title}</Text>
      </View>
      
      <View className="my-component__content">
        {/* 内容区域 */}
      </View>
      
      <View className="my-component__footer">
        <Button type="default" onClick={onCancel}>
          取消
        </Button>
        <Button type="primary" onClick={handleClick}>
          确认
        </Button>
      </View>
    </View>
  );
};

export default MyComponent;
```

#### 样式模板（SCSS）
```scss
// variables
$component-primary-color: #fa541c;
$component-bg-color: #fff;
$component-border-radius: 8px;

.my-component {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  
  &__header {
    padding: 20px;
    border-bottom: 1px solid #eee;
    
    &--active {
      background-color: $component-primary-color;
    }
  }
  
  &__title {
    font-size: 32px;
    color: #333;
    font-weight: bold;
  }
  
  &__content {
    flex: 1;
    padding: 20px;
    overflow-y: auto;
  }
  
  &__footer {
    display: flex;
    gap: 20px;
    padding: 20px;
    border-top: 1px solid #eee;
    
    :nest(.nut-button) {
      flex: 1;
    }
  }
}
```

### 3. 使用 NutUI 组件的最佳实践

#### Form 表单组件
```typescript
import React, { useState } from 'react';
import { Form, Input, Button, Radio, Checkbox } from '@nutui/nutui-react-taro';
import Taro from '@tarojs/taro';

interface FormData {
  username: string;
  password: string;
  remember: boolean;
  agree: boolean;
}

const LoginForm = () => {
  const [formData, setFormData] = useState<FormData>({
    username: '',
    password: '',
    remember: false,
    agree: false,
  });
  
  const handleSubmit = async () => {
    // 表单验证
    if (!formData.username || !formData.password) {
      Taro.showToast({ title: '请填写完整信息', icon: 'none' });
      return;
    }
    
    if (!formData.agree) {
      Taro.showToast({ title: '请同意协议', icon: 'none' });
      return;
    }
    
    try {
      // 提交逻辑
      await loginApi(formData);
      Taro.showToast({ title: '登录成功', icon: 'success' });
    } catch (error) {
      Taro.showToast({ title: '登录失败', icon: 'error' });
    }
  };
  
  return (
    <Form className="login-form">
      <Form.Item label="用户名" name="username">
        <Input
          placeholder="请输入用户名"
          value={formData.username}
          onChange={(value) => setFormData({ ...formData, username: value })}
          clearable
        />
      </Form.Item>
      
      <Form.Item label="密码" name="password">
        <Input
          placeholder="请输入密码"
          type="password"
          value={formData.password}
          onChange={(value) => setFormData({ ...formData, password: value })}
        />
      </Form.Item>
      
      <Form.Item label="记住我" name="remember">
        <Radio
          checked={formData.remember}
          onChange={(checked) => setFormData({ ...formData, remember: checked })}
        >
          下次自动登录
        </Radio>
      </Form.Item>
      
      <Form.Item label="" name="agree">
        <Checkbox
          checked={formData.agree}
          onChange={(checked) => setFormData({ ...formData, agree: checked })}
        >
          我已阅读并同意《用户协议》
        </Checkbox>
      </Form.Item>
      
      <Button type="primary" block onClick={handleSubmit}>
        登录
      </Button>
    </Form>
  );
};

export default LoginForm;
```

#### Dialog 弹窗组件
```typescript
import React, { useState } from 'react';
import { Dialog, Button, Toast } from '@nutui/nutui-react-taro';

const MyDialog = () => {
  const [visible, setVisible] = useState(false);
  const [loading, setLoading] = useState(false);
  
  const handleConfirm = async () => {
    setLoading(true);
    try {
      // 异步操作
      await new Promise(resolve => setTimeout(resolve, 1000));
      Toast.show('操作成功');
      setVisible(false);
    } catch (error) {
      Toast.show('操作失败');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <>
      <Button type="primary" onClick={() => setVisible(true)}>
        打开弹窗
      </Button>
      
      <Dialog
        visible={visible}
        title="提示"
        content="这是一段提示信息，确认后将执行某个操作。"
        confirmText="确定"
        cancelText="取消"
        loading={loading}
        onConfirm={handleConfirm}
        onCancel={() => setVisible(false)}
      />
    </>
  );
};

export default MyDialog;
```

#### Picker 选择器组件
```typescript
import React, { useState } from 'react';
import { Picker, Cell, Toast } from '@nutui/nutui-react-taro';
import Taro from '@tarojs/taro';

const MyPicker = () => {
  const [showPicker, setShowPicker] = useState(false);
  const [selectedValue, setSelectedValue] = useState('');
  
  const columns = [
    { text: '北京', value: 'beijing' },
    { text: '上海', value: 'shanghai' },
    { text: '广州', value: 'guangzhou' },
    { text: '深圳', value: 'shenzhen' },
  ];
  
  const handleConfirm = ({ selectedOptions }: any) => {
    setSelectedValue(selectedOptions[0].value);
    setShowPicker(false);
    Toast.show(`选择了：${selectedOptions[0].text}`);
  };
  
  return (
    <>
      <Cell
        title="选择城市"
        isLink
        description={selectedValue || '请选择'}
        onClick={() => setShowPicker(true)}
      />
      
      <Picker
        visible={showPicker}
        columns={columns}
        onConfirm={handleConfirm}
        onCancel={() => setShowPicker(false)}
        onClose={() => setShowPicker(false)}
      />
    </>
  );
};

export default MyPicker;
```

#### Swiper 轮播组件
```typescript
import React, { useState } from 'react';
import { Swiper as NutSwiper, SwiperItem } from '@nutui/nutui-react-taro';
import { Image } from '@tarojs/components';

interface Banner {
  id: string;
  imageUrl: string;
  linkUrl?: string;
}

interface IBannerSwiperProps {
  banners: Banner[];
  onItemClick?: (banner: Banner) => void;
}

const BannerSwiper: FC<IBannerSwiperProps> = ({ banners, onItemClick }) => {
  const [currentPage, setCurrentPage] = useState(0);
  
  const handleChange = (index: number) => {
    setCurrentPage(index);
  };
  
  const handleClick = (banner: Banner) => {
    if (banner.linkUrl) {
      Taro.navigateTo({ url: banner.linkUrl });
    }
    onItemClick?.(banner);
  };
  
  return (
    <View className="banner-swiper">
      <NutSwiper
        paginationColor="#fa541c"
        autoPlay
        interval={3000}
        loop
        onChange={handleChange}
      >
        {banners.map((banner) => (
          <SwiperItem key={banner.id}>
            <View onClick={() => handleClick(banner)}>
              <Image
                src={banner.imageUrl}
                mode="aspectFill"
                className="banner-image"
              />
            </View>
          </SwiperItem>
        ))}
      </NutSwiper>
      
      {/* 自定义分页指示器 */}
      <View className="pagination">
        {banners.map((_, index) => (
          <View
            key={index}
            className={`dot ${index === currentPage ? 'active' : ''}`}
          />
        ))}
      </View>
    </View>
  );
};

export default BannerSwiper;
```

### 4. 列表组件开发

#### 基础列表示例
```typescript
import React, { useState, useEffect } from 'react';
import { ScrollView, View, Text, Image } from '@tarojs/components';
import { Cell, Skeleton, Empty, PullRefresh } from '@nutui/nutui-react-taro';
import Taro from '@tarojs/taro';

interface ListItem {
  id: string;
  title: string;
  description: string;
  imageUrl?: string;
  createTime: string;
}

interface IListProps {
  onLoadMore?: (page: number) => Promise<ListItem[]>;
  onRefresh?: () => Promise<ListItem[]>;
  onItemClick?: (item: ListItem) => void;
}

const ListComponent: FC<IListProps> = ({
  onLoadMore,
  onRefresh,
  onItemClick,
}) => {
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [list, setList] = useState<ListItem[]>([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  
  const loadData = async (pageNum: number, isRefresh = false) => {
    if (loading || (!hasMore && !isRefresh)) return;
    
    setLoading(true);
    
    try {
      let data: ListItem[] = [];
      
      if (isRefresh && onRefresh) {
        data = await onRefresh();
      } else if (onLoadMore) {
        data = await onLoadMore(pageNum);
      }
      
      if (data.length === 0) {
        setHasMore(false);
      }
      
      setList(isRefresh ? data : [...list, ...data]);
      setPage(isRefresh ? 2 : pageNum + 1);
    } catch (error) {
      Taro.showToast({ title: '加载失败', icon: 'error' });
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };
  
  useEffect(() => {
    loadData(1);
  }, []);
  
  const handleRefresh = async () => {
    setRefreshing(true);
    await loadData(1, true);
  };
  
  const handleLoadMore = async () => {
    await loadData(page);
  };
  
  const renderItem = (item: ListItem) => (
    <Cell
      key={item.id}
      className="list-item"
      onClick={() => onItemClick?.(item)}
    >
      <View className="list-item__content">
        {item.imageUrl && (
          <Image
            src={item.imageUrl}
            mode="aspectFill"
            className="list-item__image"
          />
        )}
        <View className="list-item__info">
          <Text className="list-item__title">{item.title}</Text>
          <Text className="list-item__desc">{item.description}</Text>
          <Text className="list-item__time">{item.createTime}</Text>
        </View>
      </View>
    </Cell>
  );
  
  return (
    <PullRefresh refreshStatus={refreshing} onRefresh={handleRefresh}>
      <ScrollView
        scrollY
        className="list-container"
        lowerThreshold={50}
        onScrollToLower={handleLoadMore}
      >
        {loading && list.length === 0 ? (
          <Skeleton rows={10} />
        ) : list.length === 0 ? (
          <Empty description="暂无数据" />
        ) : (
          <>
            {list.map(renderItem)}
            {loading && (
              <View className="loading-tip">加载中...</View>
            )}
            {!hasMore && (
              <View className="no-more-tip">没有更多了</View>
            )}
          </>
        )}
      </ScrollView>
    </PullRefresh>
  );
};

export default ListComponent;
```

### 5. 自定义 Hooks 开发

#### useListData - 列表数据管理
```typescript
import { useState, useCallback } from 'react';
import Taro from '@tarojs/taro';

interface UseListDataOptions<T> {
  apiCall: (page: number) => Promise<T[]>;
  pageSize?: number;
}

interface UseListDataReturn<T> {
  list: T[];
  loading: boolean;
  refreshing: boolean;
  hasMore: boolean;
  page: number;
  loadMore: () => Promise<void>;
  refresh: () => Promise<void>;
  reset: () => void;
  add: (item: T) => void;
  update: (id: string | number, updater: Partial<T>) => void;
  remove: (id: string | number) => void;
}

export function useListData<T extends { id: string | number }>(
  options: UseListDataOptions<T>
): UseListDataReturn<T> {
  const { apiCall, pageSize = 10 } = options;
  
  const [list, setList] = useState<T[]>([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  
  const loadData = useCallback(async (pageNum: number, isRefresh = false) => {
    if (loading || (!hasMore && !isRefresh)) return;
    
    setLoading(true);
    
    try {
      const data = await apiCall(pageNum);
      
      if (data.length < pageSize) {
        setHasMore(false);
      }
      
      if (isRefresh) {
        setList(data);
        setPage(2);
      } else {
        setList(prev => [...prev, ...data]);
        setPage(pageNum + 1);
      }
    } catch (error) {
      Taro.showToast({ title: '加载失败', icon: 'error' });
      throw error;
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  }, [apiCall, hasMore, loading, pageSize]);
  
  const loadMore = useCallback(async () => {
    await loadData(page);
  }, [loadData, page]);
  
  const refresh = useCallback(async () => {
    setRefreshing(true);
    await loadData(1, true);
  }, [loadData]);
  
  const reset = useCallback(() => {
    setList([]);
    setPage(1);
    setHasMore(true);
  }, []);
  
  const add = useCallback((item: T) => {
    setList(prev => [item, ...prev]);
  }, []);
  
  const update = useCallback((id: string | number, updater: Partial<T>) => {
    setList(prev => prev.map(item => 
      item.id === id ? { ...item, ...updater } : item
    ));
  }, []);
  
  const remove = useCallback((id: string | number) => {
    setList(prev => prev.filter(item => item.id !== id));
  }, []);
  
  return {
    list,
    loading,
    refreshing,
    hasMore,
    page,
    loadMore,
    refresh,
    reset,
    add,
    update,
    remove,
  };
}
```

#### useModal - 弹窗管理
```typescript
import { useState, useCallback } from 'react';

interface UseModalOptions {
  defaultVisible?: boolean;
  onOpen?: () => void;
  onClose?: () => void;
}

interface UseModalReturn {
  visible: boolean;
  open: () => void;
  close: () => void;
  toggle: () => void;
}

export function useModal(options: UseModalOptions = {}): UseModalReturn {
  const { defaultVisible = false, onOpen, onClose } = options;
  
  const [visible, setVisible] = useState(defaultVisible);
  
  const open = useCallback(() => {
    setVisible(true);
    onOpen?.();
  }, [onOpen]);
  
  const close = useCallback(() => {
    setVisible(false);
    onClose?.();
  }, [onClose]);
  
  const toggle = useCallback(() => {
    if (visible) {
      close();
    } else {
      open();
    }
  }, [visible, open, close]);
  
  return {
    visible,
    open,
    close,
    toggle,
  };
}
```

#### useForm - 表单管理
```typescript
import { useState, useCallback } from 'react';

interface UseFormOptions<T> {
  initialValues: T;
  validate?: (values: T) => Record<string, string>;
  onSubmit: (values: T) => Promise<void>;
}

interface UseFormReturn<T> {
  values: T;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  loading: boolean;
  setValues: (newValues: Partial<T>) => void;
  setFieldValue: (field: keyof T, value: any) => void;
  handleBlur: (field: keyof T) => void;
  handleSubmit: () => Promise<boolean>;
  reset: () => void;
  validate: () => boolean;
}

export function useForm<T extends Record<string, any>>(
  options: UseFormOptions<T>
): UseFormReturn<T> {
  const { initialValues, validate: validateFn, onSubmit } = options;
  
  const [values, setValuesState] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});
  const [loading, setLoading] = useState(false);
  
  const setValues = useCallback((newValues: Partial<T>) => {
    setValuesState(prev => ({ ...prev, ...newValues }));
  }, []);
  
  const setFieldValue = useCallback((field: keyof T, value: any) => {
    setValuesState(prev => ({ ...prev, [field]: value }));
  }, []);
  
  const handleBlur = useCallback((field: keyof T) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  }, []);
  
  const validate = useCallback(() => {
    if (!validateFn) return true;
    
    const validationErrors = validateFn(values);
    setErrors(validationErrors);
    return Object.keys(validationErrors).length === 0;
  }, [validateFn, values]);
  
  const handleSubmit = useCallback(async () => {
    // 标记所有字段为已触摸
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {} as Record<string, boolean>);
    setTouched(allTouched);
    
    // 验证
    if (validateFn && !validate()) {
      return false;
    }
    
    setLoading(true);
    try {
      await onSubmit(values);
      return true;
    } catch (error) {
      return false;
    } finally {
      setLoading(false);
    }
  }, [values, validate, validateFn, onSubmit]);
  
  const reset = useCallback(() => {
    setValuesState(initialValues);
    setErrors({});
    setTouched({});
    setLoading(false);
  }, [initialValues]);
  
  return {
    values,
    errors,
    touched,
    loading,
    setValues,
    setFieldValue,
    handleBlur,
    handleSubmit,
    reset,
    validate,
  };
}
```

### 6. 组件性能优化

#### React.memo 优化
```typescript
import React, { memo, useMemo } from 'react';

interface IListItemProps {
  item: ListItem;
  onPress: (item: ListItem) => void;
}

// 使用 memo 避免不必要的重新渲染
const ListItem: React.FC<IListItemProps> = memo(({ item, onPress }) => {
  const handleClick = useCallback(() => {
    onPress(item);
  }, [item, onPress]);
  
  return (
    <Cell onClick={handleClick}>
      <Text>{item.title}</Text>
    </Cell>
  );
}, (prevProps, nextProps) => {
  // 自定义比较函数
  return prevProps.item.id === nextProps.item.id && 
         prevProps.onPress === nextProps.onPress;
});

// 列表组件中使用
const List = ({ items }) => {
  const handlePress = useCallback((item) => {
    console.log('Pressed:', item);
  }, []);
  
  return (
    <View>
      {items.map(item => (
        <ListItem 
          key={item.id} 
          item={item} 
          onPress={handlePress}
        />
      ))}
    </View>
  );
};
```

#### 使用 useMemo 缓存计算结果
```typescript
import React, { useMemo } from 'react';

const ExpensiveComponent = ({ data }) => {
  // 缓存过滤后的数据
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter(item => item.active);
  }, [data]);
  
  // 缓存排序后的数据
  const sortedData = useMemo(() => {
    console.log('Sorting data...');
    return [...filteredData].sort((a, b) => a.priority - b.priority);
  }, [filteredData]);
  
  return (
    <View>
      {sortedData.map(item => (
        <Text key={item.id}>{item.name}</Text>
      ))}
    </View>
  );
};
```

### 7. 组件文档示例

```markdown
# MyComponent 组件文档

## 功能描述
用于 XXX 场景的功能组件

## Props 属性

| 属性 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| title | string | 是 | - | 组件标题 |
| visible | boolean | 是 | false | 是否显示 |
| onConfirm | () => void | 否 | - | 确认回调 |
| onCancel | () => void | 否 | - | 取消回调 |

## Events 事件

| 事件名 | 回调参数 | 说明 |
|--------|----------|------|
| onConfirm | - | 点击确认按钮触发 |
| onCancel | - | 点击取消按钮触发 |

## Slots 插槽

| 插槽名 | 说明 |
|--------|------|
| header | 头部自定义内容 |
| footer | 底部自定义内容 |

## 使用示例

```tsx
<MyComponent
  title="示例标题"
  visible={true}
  onConfirm={() => console.log('确认')}
  onCancel={() => console.log('取消')}
/>
```

## 注意事项
1. 组件需要配合 xxx 使用
2. 注意 xxx 场景的性能问题
```

## 组件测试检查清单

- [ ] TypeScript 类型定义完整
- [ ] Props 有默认值或明确标注必填
- [ ] 样式隔离，不影响其他组件
- [ ] 支持主流机型适配
- [ ] 边界情况处理（空数据、加载失败等）
- [ ] 内存泄漏检查（清理定时器、监听器等）
- [ ] 无障碍访问支持
- [ ] 性能优化（memo、useMemo 等）
