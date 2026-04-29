# CrushOn.AI 支付系统组件设计文档

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**文档类型**: 组件设计文档  
**技术栈**: React + TypeScript  
**设计系统**: 基于样式设计规范

## 一、组件架构概览

### 1.1 组件层级结构

```
PaymentSystem/
├── containers/          # 容器组件
│   ├── PaymentFlow     # 支付流程主容器
│   ├── CheckoutPage    # 结账页面容器
│   └── SubscriptionManager # 订阅管理容器
├── components/         # 业务组件
│   ├── PlanSelector    # 套餐选择组件
│   ├── PaymentMethods  # 支付方式组件
│   ├── PaymentForm     # 支付表单组件
│   ├── PaymentStatus   # 支付状态组件
│   └── OrderSummary    # 订单摘要组件
├── ui/                 # 基础UI组件
│   ├── Button          # 按钮组件
│   ├── Input           # 输入框组件
│   ├── Card            # 卡片组件
│   ├── Modal           # 模态框组件
│   ├── Progress        # 进度条组件
│   └── Toast           # 提示组件
└── hooks/              # 自定义Hooks
    ├── usePayment      # 支付逻辑Hook
    ├── useValidation   # 表单验证Hook
    └── useAnalytics    # 数据分析Hook
```

### 1.2 组件设计原则

| 原则 | 描述 | 实现方式 |
|-----|------|----------|
| **单一职责** | 每个组件只负责一个功能 | 功能拆分、组件细化 |
| **可复用性** | 组件可在不同场景复用 | Props配置、插槽设计 |
| **可测试性** | 组件易于单元测试 | 纯函数、依赖注入 |
| **可访问性** | 支持键盘和屏幕阅读器 | ARIA属性、语义化HTML |
| **性能优化** | 避免不必要的重渲染 | React.memo、useMemo |

## 二、核心业务组件

### 2.1 PlanSelector 套餐选择组件

#### 组件说明
负责展示可选套餐、处理套餐选择、优惠码验证等功能

#### 组件接口

```typescript
interface PlanSelectorProps {
  // 数据属性
  plans: Plan[];                    // 可选套餐列表
  currentPlan?: Plan;               // 当前套餐(升级场景)
  recommendedPlanId?: string;       // 推荐套餐ID
  defaultBillingCycle?: BillingCycle; // 默认计费周期
  
  // 功能属性
  showComparison?: boolean;         // 是否显示对比
  allowPromoCode?: boolean;         // 是否允许优惠码
  
  // 事件回调
  onPlanSelect: (plan: Plan, billingCycle: BillingCycle) => void;
  onPromoCodeApply?: (code: string) => Promise<PromoValidation>;
  onBillingCycleChange?: (cycle: BillingCycle) => void;
  
  // 样式属性
  layout?: 'horizontal' | 'vertical' | 'grid';
  className?: string;
}

interface Plan {
  id: string;
  name: string;
  description: string;
  features: string[];
  pricing: {
    monthly: number;
    quarterly?: number;
    yearly: number;
  };
  badge?: 'recommended' | 'popular' | 'best-value';
  disabled?: boolean;
}

type BillingCycle = 'monthly' | 'quarterly' | 'yearly';

interface PromoValidation {
  isValid: boolean;
  discount?: number;
  message?: string;
}
```

#### 组件实现

```tsx
import React, { useState, useCallback, useMemo } from 'react';
import { Card, Button, Input, Badge } from '@/ui';
import { formatPrice, calculateSavings } from '@/utils';
import { useDebounce } from '@/hooks';

export const PlanSelector: React.FC<PlanSelectorProps> = ({
  plans,
  currentPlan,
  recommendedPlanId,
  defaultBillingCycle = 'monthly',
  showComparison = true,
  allowPromoCode = false,
  onPlanSelect,
  onPromoCodeApply,
  onBillingCycleChange,
  layout = 'horizontal',
  className
}) => {
  const [selectedPlanId, setSelectedPlanId] = useState<string>(
    recommendedPlanId || plans[0]?.id
  );
  const [billingCycle, setBillingCycle] = useState<BillingCycle>(
    defaultBillingCycle
  );
  const [promoCode, setPromoCode] = useState('');
  const [promoValidation, setPromoValidation] = useState<PromoValidation | null>(null);
  const [isValidating, setIsValidating] = useState(false);
  
  // 防抖优惠码验证
  const debouncedPromoCode = useDebounce(promoCode, 500);
  
  // 优惠码验证
  useEffect(() => {
    if (debouncedPromoCode && onPromoCodeApply) {
      validatePromoCode(debouncedPromoCode);
    }
  }, [debouncedPromoCode]);
  
  const validatePromoCode = async (code: string) => {
    setIsValidating(true);
    try {
      const validation = await onPromoCodeApply(code);
      setPromoValidation(validation);
    } catch (error) {
      setPromoValidation({
        isValid: false,
        message: '优惠码验证失败'
      });
    } finally {
      setIsValidating(false);
    }
  };
  
  // 计算最终价格
  const calculateFinalPrice = useCallback((plan: Plan) => {
    let price = plan.pricing[billingCycle];
    
    // 应用优惠
    if (promoValidation?.isValid && promoValidation.discount) {
      price = price * (1 - promoValidation.discount / 100);
    }
    
    return price;
  }, [billingCycle, promoValidation]);
  
  // 处理套餐选择
  const handlePlanSelect = (plan: Plan) => {
    setSelectedPlanId(plan.id);
    onPlanSelect(plan, billingCycle);
  };
  
  // 处理计费周期切换
  const handleBillingCycleChange = (cycle: BillingCycle) => {
    setBillingCycle(cycle);
    onBillingCycleChange?.(cycle);
  };
  
  // 计算节省金额
  const savings = useMemo(() => {
    if (billingCycle === 'yearly') {
      return plans.map(plan => ({
        planId: plan.id,
        amount: plan.pricing.monthly * 12 - plan.pricing.yearly
      }));
    }
    return [];
  }, [plans, billingCycle]);
  
  return (
    <div className={`plan-selector ${layout} ${className}`}>
      {/* 计费周期切换 */}
      <div className="billing-cycle-selector">
        <div className="cycle-tabs">
          {['monthly', 'quarterly', 'yearly'].map(cycle => (
            <button
              key={cycle}
              className={`cycle-tab ${billingCycle === cycle ? 'active' : ''}`}
              onClick={() => handleBillingCycleChange(cycle as BillingCycle)}
            >
              {cycle === 'monthly' && '月付'}
              {cycle === 'quarterly' && '季付'}
              {cycle === 'yearly' && '年付'}
              {cycle === 'yearly' && savings.length > 0 && (
                <Badge variant="success" size="small">
                  省{Math.round(savings[0].amount / (plans[0].pricing.monthly * 12) * 100)}%
                </Badge>
              )}
            </button>
          ))}
        </div>
      </div>
      
      {/* 套餐卡片列表 */}
      <div className={`plan-cards ${layout}`}>
        {plans.map(plan => {
          const isRecommended = plan.id === recommendedPlanId;
          const isSelected = plan.id === selectedPlanId;
          const finalPrice = calculateFinalPrice(plan);
          const saving = savings.find(s => s.planId === plan.id);
          
          return (
            <Card
              key={plan.id}
              className={`plan-card ${isSelected ? 'selected' : ''} ${isRecommended ? 'recommended' : ''}`}
              onClick={() => !plan.disabled && handlePlanSelect(plan)}
            >
              {/* 推荐标签 */}
              {plan.badge && (
                <Badge
                  variant={plan.badge === 'recommended' ? 'primary' : 'secondary'}
                  className="plan-badge"
                >
                  {plan.badge === 'recommended' && '推荐'}
                  {plan.badge === 'popular' && '最受欢迎'}
                  {plan.badge === 'best-value' && '最超值'}
                </Badge>
              )}
              
              {/* 套餐信息 */}
              <div className="plan-header">
                <h3 className="plan-name">{plan.name}</h3>
                <p className="plan-description">{plan.description}</p>
              </div>
              
              {/* 价格展示 */}
              <div className="plan-price">
                {promoValidation?.isValid && (
                  <span className="original-price">
                    ¥{plan.pricing[billingCycle]}
                  </span>
                )}
                <span className="price-amount">
                  <span className="currency">¥</span>
                  <span className="amount">{Math.floor(finalPrice)}</span>
                  <span className="decimal">.{Math.round((finalPrice % 1) * 100)}</span>
                </span>
                <span className="price-period">
                  /{billingCycle === 'monthly' ? '月' : billingCycle === 'quarterly' ? '季' : '年'}
                </span>
              </div>
              
              {/* 节省提示 */}
              {saving && saving.amount > 0 && (
                <div className="saving-tip">
                  相比月付节省 ¥{saving.amount.toFixed(2)}
                </div>
              )}
              
              {/* 功能列表 */}
              <ul className="plan-features">
                {plan.features.map((feature, index) => (
                  <li key={index} className="feature-item">
                    <svg className="feature-icon" width="16" height="16">
                      <use href="#icon-check"></use>
                    </svg>
                    <span>{feature}</span>
                  </li>
                ))}
              </ul>
              
              {/* 选择按钮 */}
              <Button
                variant={isRecommended ? 'primary' : 'secondary'}
                size="large"
                fullWidth
                disabled={plan.disabled}
                className="plan-select-btn"
              >
                {isSelected ? '已选择' : '选择此套餐'}
              </Button>
              
              {/* 升级/降级提示 */}
              {currentPlan && (
                <div className="plan-change-tip">
                  {plan.pricing[billingCycle] > currentPlan.pricing[billingCycle] ? (
                    <span className="upgrade">升级套餐</span>
                  ) : plan.pricing[billingCycle] < currentPlan.pricing[billingCycle] ? (
                    <span className="downgrade">降级套餐(下个周期生效)</span>
                  ) : null}
                </div>
              )}
            </Card>
          );
        })}
      </div>
      
      {/* 优惠码输入 */}
      {allowPromoCode && (
        <div className="promo-code-section">
          <Input
            label="优惠码"
            placeholder="请输入优惠码"
            value={promoCode}
            onChange={(e) => setPromoCode(e.target.value)}
            suffix={
              isValidating ? (
                <Spinner size="small" />
              ) : promoValidation?.isValid ? (
                <Icon name="check" color="success" />
              ) : promoValidation ? (
                <Icon name="x" color="error" />
              ) : null
            }
            error={promoValidation && !promoValidation.isValid}
            helperText={promoValidation?.message}
          />
        </div>
      )}
      
      {/* 套餐对比表 */}
      {showComparison && (
        <div className="plan-comparison">
          <Button
            variant="text"
            onClick={() => setShowComparisonModal(true)}
          >
            查看详细功能对比
          </Button>
        </div>
      )}
    </div>
  );
};
```

### 2.2 PaymentMethods 支付方式组件

#### 组件说明
展示可用支付方式，智能推荐最优选项，处理支付方式选择

#### 组件接口

```typescript
interface PaymentMethodsProps {
  // 数据属性
  methods: PaymentMethod[];         // 可用支付方式
  savedMethods?: SavedPaymentMethod[]; // 已保存的支付方式
  recommendedMethodId?: string;     // 推荐的支付方式
  
  // 功能属性
  allowSaveMethod?: boolean;         // 是否允许保存支付方式
  showSuccessRate?: boolean;         // 是否显示成功率
  
  // 事件回调
  onMethodSelect: (method: PaymentMethod) => void;
  onSavedMethodSelect?: (method: SavedPaymentMethod) => void;
  onMethodRemove?: (methodId: string) => Promise<void>;
  
  // 样式属性
  layout?: 'list' | 'grid';
  className?: string;
}

interface PaymentMethod {
  id: string;
  name: string;
  type: 'credit_card' | 'debit_card' | 'paypal' | 'crypto' | 'alipay' | 'wechat';
  icon: string;
  description?: string;
  successRate?: number;
  processingTime?: string;
  available: boolean;
  badge?: 'recommended' | 'fast' | 'secure';
}

interface SavedPaymentMethod extends PaymentMethod {
  last4?: string;
  expiryDate?: string;
  isDefault?: boolean;
}
```

#### 组件实现

```tsx
export const PaymentMethods: React.FC<PaymentMethodsProps> = ({
  methods,
  savedMethods = [],
  recommendedMethodId,
  allowSaveMethod = true,
  showSuccessRate = true,
  onMethodSelect,
  onSavedMethodSelect,
  onMethodRemove,
  layout = 'list',
  className
}) => {
  const [selectedMethodId, setSelectedMethodId] = useState<string>(
    recommendedMethodId || methods[0]?.id
  );
  const [isRemovingMethod, setIsRemovingMethod] = useState<string | null>(null);
  
  // 智能排序支付方式
  const sortedMethods = useMemo(() => {
    return [...methods].sort((a, b) => {
      // 推荐方式优先
      if (a.id === recommendedMethodId) return -1;
      if (b.id === recommendedMethodId) return 1;
      
      // 成功率排序
      if (a.successRate && b.successRate) {
        return b.successRate - a.successRate;
      }
      
      return 0;
    });
  }, [methods, recommendedMethodId]);
  
  // 处理支付方式选择
  const handleMethodSelect = (method: PaymentMethod) => {
    setSelectedMethodId(method.id);
    onMethodSelect(method);
  };
  
  // 处理已保存支付方式选择
  const handleSavedMethodSelect = (method: SavedPaymentMethod) => {
    setSelectedMethodId(method.id);
    onSavedMethodSelect?.(method);
  };
  
  // 移除已保存的支付方式
  const handleMethodRemove = async (methodId: string) => {
    if (!onMethodRemove) return;
    
    setIsRemovingMethod(methodId);
    try {
      await onMethodRemove(methodId);
    } finally {
      setIsRemovingMethod(null);
    }
  };
  
  return (
    <div className={`payment-methods ${layout} ${className}`}>
      {/* 已保存的支付方式 */}
      {savedMethods.length > 0 && (
        <div className="saved-methods-section">
          <h3 className="section-title">已保存的支付方式</h3>
          <div className="saved-methods-list">
            {savedMethods.map(method => (
              <Card
                key={method.id}
                className={`saved-method-card ${selectedMethodId === method.id ? 'selected' : ''}`}
                onClick={() => handleSavedMethodSelect(method)}
              >
                <div className="method-content">
                  <img src={method.icon} alt={method.name} className="method-icon" />
                  <div className="method-info">
                    <div className="method-name">
                      {method.name}
                      {method.isDefault && (
                        <Badge size="small" variant="primary">默认</Badge>
                      )}
                    </div>
                    <div className="method-details">
                      {method.last4 && `•••• ${method.last4}`}
                      {method.expiryDate && ` · ${method.expiryDate}`}
                    </div>
                  </div>
                  <div className="method-actions">
                    {selectedMethodId === method.id && (
                      <Icon name="check-circle" color="primary" />
                    )}
                    <Button
                      variant="text"
                      size="small"
                      onClick={(e) => {
                        e.stopPropagation();
                        handleMethodRemove(method.id);
                      }}
                      loading={isRemovingMethod === method.id}
                    >
                      移除
                    </Button>
                  </div>
                </div>
              </Card>
            ))}
          </div>
        </div>
      )}
      
      {/* 其他支付方式 */}
      <div className="available-methods-section">
        <h3 className="section-title">
          {savedMethods.length > 0 ? '其他支付方式' : '选择支付方式'}
        </h3>
        <div className={`methods-${layout}`}>
          {sortedMethods.map(method => (
            <Card
              key={method.id}
              className={`method-card ${selectedMethodId === method.id ? 'selected' : ''} ${!method.available ? 'disabled' : ''}`}
              onClick={() => method.available && handleMethodSelect(method)}
            >
              {/* 推荐标签 */}
              {method.badge && (
                <Badge
                  variant={method.badge === 'recommended' ? 'primary' : 'secondary'}
                  className="method-badge"
                >
                  {method.badge === 'recommended' && '推荐'}
                  {method.badge === 'fast' && '快速'}
                  {method.badge === 'secure' && '安全'}
                </Badge>
              )}
              
              <div className="method-content">
                <img src={method.icon} alt={method.name} className="method-icon" />
                <div className="method-info">
                  <div className="method-name">{method.name}</div>
                  {method.description && (
                    <div className="method-description">{method.description}</div>
                  )}
                  
                  {/* 成功率和处理时间 */}
                  <div className="method-stats">
                    {showSuccessRate && method.successRate && (
                      <span className="success-rate">
                        <Icon name="trending-up" size="small" />
                        成功率 {method.successRate}%
                      </span>
                    )}
                    {method.processingTime && (
                      <span className="processing-time">
                        <Icon name="clock" size="small" />
                        {method.processingTime}
                      </span>
                    )}
                  </div>
                </div>
                
                {/* 选中状态 */}
                <div className="method-select">
                  {selectedMethodId === method.id ? (
                    <Icon name="check-circle" color="primary" size="large" />
                  ) : (
                    <div className="select-circle" />
                  )}
                </div>
              </div>
              
              {/* 不可用提示 */}
              {!method.available && (
                <div className="unavailable-overlay">
                  <span>暂不可用</span>
                </div>
              )}
            </Card>
          ))}
        </div>
      </div>
    </div>
  );
};
```

### 2.3 PaymentForm 支付表单组件

#### 组件说明
处理支付信息输入、实时验证、表单提交

#### 组件接口

```typescript
interface PaymentFormProps {
  // 数据属性
  paymentMethod: PaymentMethod;     // 选中的支付方式
  initialValues?: PaymentFormData;  // 初始值
  
  // 功能属性
  autoFocus?: boolean;               // 自动聚焦
  saveMethod?: boolean;              // 是否保存支付方式
  
  // 事件回调
  onSubmit: (data: PaymentFormData) => Promise<void>;
  onChange?: (data: Partial<PaymentFormData>) => void;
  onValidationChange?: (isValid: boolean) => void;
  
  // 样式属性
  className?: string;
}

interface PaymentFormData {
  cardNumber: string;
  cardholderName: string;
  expiryDate: string;
  cvv: string;
  billingAddress?: {
    country: string;
    postalCode: string;
  };
  saveMethod?: boolean;
}
```

#### 组件实现

```tsx
export const PaymentForm: React.FC<PaymentFormProps> = ({
  paymentMethod,
  initialValues,
  autoFocus = true,
  saveMethod = false,
  onSubmit,
  onChange,
  onValidationChange,
  className
}) => {
  const [formData, setFormData] = useState<PaymentFormData>(
    initialValues || {
      cardNumber: '',
      cardholderName: '',
      expiryDate: '',
      cvv: '',
      saveMethod: false
    }
  );
  
  const [errors, setErrors] = useState<Partial<Record<keyof PaymentFormData, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof PaymentFormData, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [cardType, setCardType] = useState<string | null>(null);
  
  // 表单验证
  const validate = useCallback((data: PaymentFormData) => {
    const newErrors: typeof errors = {};
    
    // 卡号验证
    if (!data.cardNumber) {
      newErrors.cardNumber = '请输入卡号';
    } else if (!validateCardNumber(data.cardNumber)) {
      newErrors.cardNumber = '卡号无效';
    }
    
    // 持卡人姓名验证
    if (!data.cardholderName) {
      newErrors.cardholderName = '请输入持卡人姓名';
    }
    
    // 有效期验证
    if (!data.expiryDate) {
      newErrors.expiryDate = '请输入有效期';
    } else if (!validateExpiryDate(data.expiryDate)) {
      newErrors.expiryDate = '有效期无效或已过期';
    }
    
    // CVV验证
    if (!data.cvv) {
      newErrors.cvv = '请输入CVV';
    } else if (!validateCVV(data.cvv, cardType)) {
      newErrors.cvv = 'CVV无效';
    }
    
    return newErrors;
  }, [cardType]);
  
  // 实时验证
  useEffect(() => {
    const newErrors = validate(formData);
    setErrors(newErrors);
    
    const isValid = Object.keys(newErrors).length === 0;
    onValidationChange?.(isValid);
  }, [formData, validate, onValidationChange]);
  
  // 处理输入变化
  const handleChange = (field: keyof PaymentFormData, value: string | boolean) => {
    const newData = { ...formData, [field]: value };
    
    // 特殊处理
    if (field === 'cardNumber') {
      // 格式化卡号
      newData.cardNumber = formatCardNumber(value as string);
      // 检测卡类型
      setCardType(detectCardType(value as string));
    } else if (field === 'expiryDate') {
      // 格式化有效期
      newData.expiryDate = formatExpiryDate(value as string);
    }
    
    setFormData(newData);
    onChange?.(newData);
  };
  
  // 处理焦点
  const handleBlur = (field: keyof PaymentFormData) => {
    setTouched({ ...touched, [field]: true });
  };
  
  // 提交表单
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // 标记所有字段已触碰
    const allTouched = Object.keys(formData).reduce((acc, key) => ({
      ...acc,
      [key]: true
    }), {});
    setTouched(allTouched);
    
    // 验证
    const validationErrors = validate(formData);
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    setIsSubmitting(true);
    try {
      await onSubmit(formData);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form className={`payment-form ${className}`} onSubmit={handleSubmit}>
      {/* 信用卡表单 */}
      {paymentMethod.type === 'credit_card' && (
        <>
          {/* 卡号输入 */}
          <div className="form-group">
            <Input
              label="卡号"
              type="text"
              inputMode="numeric"
              placeholder="1234 5678 9012 3456"
              value={formData.cardNumber}
              onChange={(e) => handleChange('cardNumber', e.target.value)}
              onBlur={() => handleBlur('cardNumber')}
              error={touched.cardNumber && !!errors.cardNumber}
              helperText={touched.cardNumber && errors.cardNumber}
              maxLength={19}
              autoFocus={autoFocus}
              prefix={
                cardType && (
                  <img
                    src={`/icons/cards/${cardType}.svg`}
                    alt={cardType}
                    className="card-type-icon"
                  />
                )
              }
            />
          </div>
          
          {/* 有效期和CVV */}
          <div className="form-row">
            <div className="form-group">
              <Input
                label="有效期"
                type="text"
                inputMode="numeric"
                placeholder="MM/YY"
                value={formData.expiryDate}
                onChange={(e) => handleChange('expiryDate', e.target.value)}
                onBlur={() => handleBlur('expiryDate')}
                error={touched.expiryDate && !!errors.expiryDate}
                helperText={touched.expiryDate && errors.expiryDate}
                maxLength={5}
              />
            </div>
            
            <div className="form-group">
              <Input
                label="CVV"
                type="text"
                inputMode="numeric"
                placeholder="123"
                value={formData.cvv}
                onChange={(e) => handleChange('cvv', e.target.value)}
                onBlur={() => handleBlur('cvv')}
                error={touched.cvv && !!errors.cvv}
                helperText={touched.cvv && errors.cvv}
                maxLength={cardType === 'amex' ? 4 : 3}
                suffix={
                  <Tooltip content="卡片背面的3位数字">
                    <Icon name="help-circle" size="small" />
                  </Tooltip>
                }
              />
            </div>
          </div>
          
          {/* 持卡人姓名 */}
          <div className="form-group">
            <Input
              label="持卡人姓名"
              type="text"
              placeholder="张三"
              value={formData.cardholderName}
              onChange={(e) => handleChange('cardholderName', e.target.value.toUpperCase())}
              onBlur={() => handleBlur('cardholderName')}
              error={touched.cardholderName && !!errors.cardholderName}
              helperText={touched.cardholderName && errors.cardholderName}
            />
          </div>
          
          {/* 保存支付方式 */}
          {saveMethod && (
            <div className="form-group">
              <Checkbox
                label="保存此支付方式以便下次使用"
                checked={formData.saveMethod}
                onChange={(checked) => handleChange('saveMethod', checked)}
              />
            </div>
          )}
        </>
      )}
      
      {/* PayPal表单 */}
      {paymentMethod.type === 'paypal' && (
        <div className="paypal-form">
          <div className="paypal-info">
            <Icon name="info-circle" />
            <p>点击下方按钮将跳转到PayPal完成支付</p>
          </div>
          <Button
            type="submit"
            variant="primary"
            size="large"
            fullWidth
            loading={isSubmitting}
            icon={<img src="/icons/paypal.svg" alt="PayPal" />}
          >
            使用PayPal支付
          </Button>
        </div>
      )}
      
      {/* 提交按钮 */}
      {paymentMethod.type === 'credit_card' && (
        <Button
          type="submit"
          variant="primary"
          size="large"
          fullWidth
          loading={isSubmitting}
          disabled={Object.keys(errors).length > 0}
        >
          确认支付
        </Button>
      )}
      
      {/* 安全提示 */}
      <div className="security-info">
        <Icon name="lock" size="small" />
        <span>您的支付信息已加密保护</span>
      </div>
    </form>
  );
};
```

## 三、基础UI组件

### 3.1 Button 按钮组件

```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'text' | 'danger';
  size?: 'small' | 'medium' | 'large';
  fullWidth?: boolean;
  loading?: boolean;
  disabled?: boolean;
  icon?: React.ReactNode;
  children: React.ReactNode;
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  className?: string;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'medium',
  fullWidth = false,
  loading = false,
  disabled = false,
  icon,
  children,
  onClick,
  type = 'button',
  className
}) => {
  return (
    <button
      type={type}
      className={`
        btn 
        btn-${variant} 
        btn-${size}
        ${fullWidth ? 'btn-full' : ''}
        ${loading ? 'btn-loading' : ''}
        ${className}
      `}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {icon && !loading && <span className="btn-icon">{icon}</span>}
      {loading && <Spinner size="small" />}
      <span className="btn-text">{children}</span>
    </button>
  );
};
```

### 3.2 Input 输入框组件

```tsx
interface InputProps {
  label?: string;
  type?: string;
  placeholder?: string;
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  onBlur?: () => void;
  error?: boolean;
  helperText?: string;
  prefix?: React.ReactNode;
  suffix?: React.ReactNode;
  disabled?: boolean;
  required?: boolean;
  maxLength?: number;
  inputMode?: string;
  autoFocus?: boolean;
  className?: string;
}

export const Input: React.FC<InputProps> = ({
  label,
  type = 'text',
  placeholder,
  value,
  onChange,
  onBlur,
  error,
  helperText,
  prefix,
  suffix,
  disabled,
  required,
  maxLength,
  inputMode,
  autoFocus,
  className
}) => {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    if (autoFocus && inputRef.current) {
      inputRef.current.focus();
    }
  }, [autoFocus]);
  
  return (
    <div className={`input-group ${className}`}>
      {label && (
        <label className={`input-label ${required ? 'required' : ''}`}>
          {label}
        </label>
      )}
      
      <div className={`input-wrapper ${error ? 'error' : ''}`}>
        {prefix && <span className="input-prefix">{prefix}</span>}
        
        <input
          ref={inputRef}
          type={type}
          className="input"
          placeholder={placeholder}
          value={value}
          onChange={onChange}
          onBlur={onBlur}
          disabled={disabled}
          required={required}
          maxLength={maxLength}
          inputMode={inputMode}
          aria-invalid={error}
          aria-describedby={helperText ? 'helper-text' : undefined}
        />
        
        {suffix && <span className="input-suffix">{suffix}</span>}
      </div>
      
      {helperText && (
        <span
          id="helper-text"
          className={`input-helper ${error ? 'error' : ''}`}
        >
          {helperText}
        </span>
      )}
    </div>
  );
};
```

## 四、自定义Hooks

### 4.1 usePayment Hook

```typescript
interface UsePaymentOptions {
  onSuccess?: (result: PaymentResult) => void;
  onError?: (error: PaymentError) => void;
  retryOptions?: RetryOptions;
}

export const usePayment = (options: UsePaymentOptions = {}) => {
  const [status, setStatus] = useState<PaymentStatus>('idle');
  const [error, setError] = useState<PaymentError | null>(null);
  const [result, setResult] = useState<PaymentResult | null>(null);
  const [retryCount, setRetryCount] = useState(0);
  
  const processPayment = useCallback(async (paymentData: PaymentData) => {
    setStatus('processing');
    setError(null);
    
    try {
      // 创建支付会话
      const session = await createPaymentSession(paymentData);
      
      // 令牌化敏感数据
      const token = await tokenizePaymentData(paymentData);
      
      // 风险评估
      const riskScore = await assessRisk(session, paymentData);
      
      // 选择最优网关
      const gateway = selectOptimalGateway(riskScore, paymentData);
      
      // 执行支付
      const paymentResult = await executePayment({
        gateway,
        token,
        session,
        ...paymentData
      });
      
      setResult(paymentResult);
      setStatus('success');
      options.onSuccess?.(paymentResult);
      
      return paymentResult;
      
    } catch (err) {
      const paymentError = err as PaymentError;
      setError(paymentError);
      
      // 智能重试
      if (shouldRetry(paymentError, retryCount, options.retryOptions)) {
        setRetryCount(retryCount + 1);
        setStatus('retrying');
        
        // 延迟重试
        setTimeout(() => {
          processPayment(paymentData);
        }, getRetryDelay(retryCount));
        
      } else {
        setStatus('failed');
        options.onError?.(paymentError);
      }
      
      throw paymentError;
    }
  }, [retryCount, options]);
  
  const reset = useCallback(() => {
    setStatus('idle');
    setError(null);
    setResult(null);
    setRetryCount(0);
  }, []);
  
  return {
    processPayment,
    status,
    error,
    result,
    retryCount,
    reset
  };
};
```

### 4.2 useValidation Hook

```typescript
interface ValidationRules<T> {
  [K in keyof T]?: ValidationRule<T[K]>;
}

interface ValidationRule<T> {
  required?: boolean | string;
  pattern?: RegExp | { value: RegExp; message: string };
  validate?: (value: T) => boolean | string;
  minLength?: number | { value: number; message: string };
  maxLength?: number | { value: number; message: string };
}

export const useValidation = <T extends Record<string, any>>(
  rules: ValidationRules<T>
) => {
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  
  const validate = useCallback((data: T) => {
    const newErrors: typeof errors = {};
    
    Object.keys(rules).forEach((field) => {
      const fieldRules = rules[field as keyof T];
      const value = data[field as keyof T];
      
      if (!fieldRules) return;
      
      // Required验证
      if (fieldRules.required) {
        if (!value || value === '') {
          newErrors[field as keyof T] = 
            typeof fieldRules.required === 'string' 
              ? fieldRules.required 
              : `${field}是必填项`;
          return;
        }
      }
      
      // Pattern验证
      if (fieldRules.pattern && value) {
        const pattern = typeof fieldRules.pattern === 'object' 
          ? fieldRules.pattern.value 
          : fieldRules.pattern;
        const message = typeof fieldRules.pattern === 'object' 
          ? fieldRules.pattern.message 
          : `${field}格式不正确`;
          
        if (!pattern.test(value)) {
          newErrors[field as keyof T] = message;
          return;
        }
      }
      
      // 自定义验证
      if (fieldRules.validate && value) {
        const result = fieldRules.validate(value);
        if (result !== true) {
          newErrors[field as keyof T] = 
            typeof result === 'string' ? result : `${field}验证失败`;
          return;
        }
      }
    });
    
    setErrors(newErrors);
    return newErrors;
  }, [rules]);
  
  const touch = useCallback((field: keyof T) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  }, []);
  
  const reset = useCallback(() => {
    setErrors({});
    setTouched({});
  }, []);
  
  const isValid = Object.keys(errors).length === 0;
  
  return {
    errors,
    touched,
    validate,
    touch,
    reset,
    isValid
  };
};
```

## 五、组件测试规范

### 5.1 单元测试示例

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { PlanSelector } from './PlanSelector';

describe('PlanSelector', () => {
  const mockPlans = [
    {
      id: 'basic',
      name: '基础版',
      description: '适合个人用户',
      features: ['功能1', '功能2'],
      pricing: { monthly: 19, yearly: 190 }
    },
    {
      id: 'pro',
      name: '专业版',
      description: '适合团队使用',
      features: ['功能1', '功能2', '功能3'],
      pricing: { monthly: 39, yearly: 390 }
    }
  ];
  
  const mockOnPlanSelect = jest.fn();
  
  beforeEach(() => {
    mockOnPlanSelect.mockClear();
  });
  
  test('应该渲染所有套餐', () => {
    render(
      <PlanSelector
        plans={mockPlans}
        onPlanSelect={mockOnPlanSelect}
      />
    );
    
    expect(screen.getByText('基础版')).toBeInTheDocument();
    expect(screen.getByText('专业版')).toBeInTheDocument();
  });
  
  test('应该正确处理套餐选择', () => {
    render(
      <PlanSelector
        plans={mockPlans}
        onPlanSelect={mockOnPlanSelect}
      />
    );
    
    const proCard = screen.getByText('专业版').closest('.plan-card');
    fireEvent.click(proCard);
    
    expect(mockOnPlanSelect).toHaveBeenCalledWith(
      mockPlans[1],
      'monthly'
    );
  });
  
  test('应该正确切换计费周期', () => {
    render(
      <PlanSelector
        plans={mockPlans}
        onPlanSelect={mockOnPlanSelect}
      />
    );
    
    const yearlyTab = screen.getByText('年付');
    fireEvent.click(yearlyTab);
    
    // 检查价格是否更新
    expect(screen.getByText('190')).toBeInTheDocument();
  });
  
  test('应该正确验证优惠码', async () => {
    const mockPromoValidation = jest.fn().mockResolvedValue({
      isValid: true,
      discount: 20,
      message: '优惠码有效'
    });
    
    render(
      <PlanSelector
        plans={mockPlans}
        onPlanSelect={mockOnPlanSelect}
        allowPromoCode
        onPromoCodeApply={mockPromoValidation}
      />
    );
    
    const promoInput = screen.getByPlaceholderText('请输入优惠码');
    fireEvent.change(promoInput, { target: { value: 'SAVE20' } });
    
    await waitFor(() => {
      expect(mockPromoValidation).toHaveBeenCalledWith('SAVE20');
      expect(screen.getByText('优惠码有效')).toBeInTheDocument();
    });
  });
});
```

## 六、性能优化

### 6.1 组件优化策略

```tsx
// 1. 使用React.memo避免不必要的重渲染
export const PlanCard = React.memo(({ plan, selected, onSelect }) => {
  return (
    <Card onClick={() => onSelect(plan)}>
      {/* 组件内容 */}
    </Card>
  );
}, (prevProps, nextProps) => {
  // 自定义比较函数
  return prevProps.selected === nextProps.selected &&
         prevProps.plan.id === nextProps.plan.id;
});

// 2. 使用useMemo缓存计算结果
const expensiveCalculation = useMemo(() => {
  return calculateDiscountedPrice(plan, promoCode);
}, [plan.id, promoCode]);

// 3. 使用useCallback缓存函数引用
const handleSelect = useCallback((plan) => {
  onPlanSelect(plan, billingCycle);
}, [billingCycle, onPlanSelect]);

// 4. 懒加载非关键组件
const ComparisonModal = lazy(() => import('./ComparisonModal'));

// 5. 虚拟化长列表
import { FixedSizeList } from 'react-window';

const PaymentMethodsList = ({ methods }) => (
  <FixedSizeList
    height={400}
    itemCount={methods.length}
    itemSize={80}
    width="100%"
  >
    {({ index, style }) => (
      <div style={style}>
        <PaymentMethodCard method={methods[index]} />
      </div>
    )}
  </FixedSizeList>
);
```

---

**文档状态**: ✅ 已完成  
**下一步**: 开始技术实现阶段