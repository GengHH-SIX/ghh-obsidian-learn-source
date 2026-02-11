对于包含**动画组件**和**Composables**的库，测试策略需要精准分层。以下是针对这两类核心部分的测试方案、具体代码和最佳实践。

### 一、动画组件测试策略

动画组件的测试分为两个层级：**逻辑与状态**测试（在`jsdom`/`happy-dom`中进行，占80%）和**渲染与样式**测试（在真实浏览器中进行，占20%）。

#### 1.1 逻辑与状态测试（`Vitest` + `@vue/test-utils` + `happy-dom`）

这是测试的主体，速度快，能覆盖绝大部分交互逻辑。

**核心配置 (`vitest.config.ts`):**
```javascript
import { defineConfig } from 'vitest/config';
import Vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [Vue()],
  test: {
    environment: 'happy-dom', // 或 'jsdom'
    globals: true, // 方便使用 describe, it, expect 无需导入
    include: ['tests/component/**/*.test.ts'],
  },
});
```

**常用测试示例:**

*   **基础Props与渲染**
    ```javascript
    // tests/component/FadeBox.test.ts
    import { mount } from '@vue/test-utils';
    import FadeBox from '../../src/components/FadeBox.vue';
    import { describe, expect, it } from 'vitest';

    describe('FadeBox 组件', () => {
      it('应该根据 initialOpacity prop 设置初始透明度', () => {
        const wrapper = mount(FadeBox, {
          props: { initialOpacity: 0.3 }
        });
        // 断言组件内部的计算属性或数据，而非最终样式
        expect(wrapper.vm.currentStyle.opacity).toBe(0.3);
      });

      it('当 visible prop 为 false 时，应该添加隐藏类名', () => {
        const wrapper = mount(FadeBox, {
          props: { visible: false }
        });
        expect(wrapper.find('.fade-box').classes()).toContain('fade-box--hidden');
      });
    });
    ```

*   **用户交互与事件**
    ```javascript
    it('点击时应该触发 enter 动画并发出 `animation-start` 事件', async () => {
      const onStart = vi.fn();
      const wrapper = mount(FadeBox, {
        props: { onAnimationStart: onStart }
      });
      // 触发交互
      await wrapper.trigger('click');
      // 验证事件被触发
      expect(onStart).toHaveBeenCalledOnce();
      // 验证组件内部状态已改变
      expect(wrapper.vm.isAnimating).toBe(true);
    });
    ```

*   **动画生命周期钩子**
    ```javascript
    import { nextTick } from 'vue';
    it('动画结束后应该发出 `animation-end` 事件', async () => {
      const wrapper = mount(FadeBox);
      // 模拟动画结束
      wrapper.vm.handleAnimationEnd();
      await nextTick(); // 等待 Vue 更新周期
      // 验证自定义事件
      expect(wrapper.emitted('animation-end')).toHaveLength(1);
    });
    ```

#### 1.2 渲染与样式保真度测试（`Vitest` 浏览器模式）

当需要验证组件在**真实浏览器中最终渲染的样式、布局或复杂CSS动画**时使用。

**专用配置 (`vitest.config.browser.ts`):**
```javascript
import { defineConfig } from 'vitest/config';
import Vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [Vue()],
  test: {
    browser: {
      enabled: true,
      provider: 'playwright',
      instances: [{ browser: 'chromium' }]
    },
    // 隔离浏览器测试文件
    include: ['tests/component-browser/**/*.test.ts'],
  },
});
```

**关键测试示例:**
```javascript
// tests/component-browser/FadeBox.style.test.ts
import { test, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import FadeBox from '../../src/components/FadeBox.vue';

test('在真实浏览器中，初始透明度应通过内联样式正确应用', async () => {
  const wrapper = await mount(FadeBox, {
    props: { initialOpacity: 0.5 }
  });
  // 获取真实DOM元素
  const el = wrapper.find('.fade-box').element;
  // 关键：使用浏览器原生的 getComputedStyle
  const computedStyle = getComputedStyle(el);
  // 断言浏览器实际计算出的值
  expect(parseFloat(computedStyle.opacity)).toBeCloseTo(0.5, 2);
});
```

### 二、Composables 测试策略

Composables（组合式函数）是纯逻辑，但可能依赖 `ref`、`onMounted` 等Vue响应式API或生命周期。使用 `@vue/test-utils` 中的 **`createTestingPinia`** （如果用到Pinia）或直接测试。

**最佳实践是：在 `Node` 环境中测试，但要模拟其依赖。**

**核心测试示例 (`tests/composables/useAnimation.test.ts`):**

```javascript
import { describe, expect, it, vi, beforeEach } from 'vitest';
// 关键：使用 @vue/test-utils 提供的测试工具来模拟组件挂载环境
import { renderHook } from '@vue/test-utils';
// 导入要测试的组合式函数
import { useSpring } from '../../src/composables/useSpring';

describe('useSpring 组合式函数', () => {
  beforeEach(() => {
    // 为每项测试重置模拟和时间
    vi.useRealTimers();
  });

  it('应该返回初始值和开始/停止方法', () => {
    // renderHook 专门用于测试 Composables
    const { result } = renderHook(() => useSpring({ target: 100 }));
    // result.current 包含组合式函数返回的所有响应式数据和方法
    expect(result.current.currentValue.value).toBe(0); // 初始值
    expect(result.current.start).toBeTypeOf('function');
    expect(result.current.stop).toBeTypeOf('function');
  });

  it('调用 start 后，currentValue 应逐渐向目标值变化', async () => {
    vi.useFakeTimers(); // 使用假计时器控制时间
    const { result } = renderHook(() => useSpring({ target: 100, stiffness: 300 }));
    
    result.current.start();
    
    // 快进时间模拟动画过程
    await vi.advanceTimersByTimeAsync(16); // 大约一帧
    const valueAfterOneFrame = result.current.currentValue.value;
    expect(valueAfterOneFrame).toBeGreaterThan(0);
    expect(valueAfterOneFrame).toBeLessThan(100);
    
    // 快进到动画应结束的时间
    await vi.advanceTimersByTimeAsync(1000);
    expect(result.current.currentValue.value).toBeCloseTo(100, 0);
  });

  it('在组件卸载时应自动停止动画', () => {
    const { result, unmount } = renderHook(() => useSpring({ target: 100 }));
    const stopSpy = vi.spyOn(result.current, 'stop');
    
    unmount(); // 模拟组件卸载
    
    expect(stopSpy).toHaveBeenCalledOnce();
  });
});
```

### 三、动画测试专用技巧与目录结构

#### 3.1 动画测试核心技巧
1.  **控制时间**：始终使用 `vi.useFakeTimers()` 和 `vi.advanceTimersByTimeAsync()` 控制动画进度，让测试在毫秒内完成。
2.  **Mock浏览器API**：对 `requestAnimationFrame`, `getComputedStyle`, `IntersectionObserver` 等进行可控的模拟。
    ```javascript
    beforeEach(() => {
      global.requestAnimationFrame = vi.fn((cb) => {
        const id = setTimeout(() => cb(performance.now()), 16);
        return id as any;
      });
    });
    ```
3.  **测试状态，而非过程**：断言动画开始/结束等**关键状态**，避免断言中间每一帧的具体数值（不稳定）。

#### 3.2 推荐的目录结构
```
src/
├── components/
│   ├── FadeBox.vue
│   └── SlideMenu.vue
├── composables/
│   ├── useSpring.ts
│   └── useTimeline.ts
└── utils/
    └── easing.ts

tests/
├── unit/                  # 纯逻辑单元测试 (Vitest + Node)
│   └── utils/
│       └── easing.test.ts
├── composables/           # Composables测试 (Vitest + renderHook)
│   ├── useSpring.test.ts
│   └── useTimeline.test.ts
├── component/             # 组件逻辑测试 (Vitest + happy-dom)
│   ├── FadeBox.test.ts
│   └── SlideMenu.test.ts
└── component-browser/     # 组件样式/渲染测试 (Vitest 浏览器模式)
    └── FadeBox.style.test.ts

vitest.config.ts           # 主配置 (指向 component/, composables/)
vitest.config.browser.ts   # 浏览器测试专用配置
playwright.config.ts       # (可选) 用于 E2E 或驱动浏览器模式
```

### 总结：你的工作流
1.  **日常开发**：在 `happy-dom` 环境中运行 `tests/composables/` 和 `tests/component/` 下的测试。它们**速度快、反馈及时**，覆盖所有逻辑。
2.  **提交/发布前**：运行浏览器模式测试 (`tests/component-browser/`)，确保关键组件在真实Chrome中的**渲染准确性**。
3.  **持续集成 (CI)**：并行运行所有测试套件。可以配置为：单元和组件测试必须通过，浏览器测试允许有少量非关键失败（因其对环境更敏感）。

如果你的某个Composable或组件有特别的**动画生命周期**（例如与`IntersectionObserver`或手势库集成），可以告诉我，我可以提供更具体的模拟和测试代码。



---
---
---

根据 **Vitest v4.0.0** 的配置要求，我为你提供完全兼容的配置文件。Vitest 4 在浏览器测试和 UI 模式上有显著变化，以下是更新后的配置：

## 📁 完整的 Vitest 4 配置文件

### 1. **主配置文件**：`vitest.config.ts`
用于单元测试、Composables测试和组件逻辑测试。

```typescript
import { defineConfig } from 'vitest/config'
import Vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    Vue()
  ],

  test: {
    // 🎯 环境配置：使用 happy-dom
    environment: 'happy-dom',
    
    // 📁 文件匹配规则（Vitest 4 语法）
    include: [
      'tests/unit/**/*.{test,spec}.ts',
      'tests/composables/**/*.{test,spec}.ts',
      'tests/component/**/*.{test,spec}.ts'
    ],
    
    exclude: [
      'tests/component-browser/**',
      'tests/e2e/**',
      '**/node_modules/**',
      '**/dist/**'
    ],
    
    // 📊 覆盖率配置（Vitest 4 内置）
    coverage: {
      enabled: true,
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      reportsDirectory: './coverage',
      exclude: [
        '**/node_modules/**',
        '**/dist/**',
        '**/*.d.ts',
        '**/*.config.*',
        'tests/**',
        'examples/**'
      ],
      thresholds: {
        statements: 80,
        branches: 70,
        functions: 80,
        lines: 80
      }
    },
    
    // ⚡ 全局和超时配置
    globals: true,
    testTimeout: 10000,
    hookTimeout: 10000,
    
    // 🧪 Mock 配置
    mockReset: true,
    clearMocks: true,
    
    // 🏃 运行配置
    pool: 'threads',
    poolOptions: {
      threads: {
        minThreads: 1,
        maxThreads: 4
      }
    },
    
    // 👁️ 报告配置
    reporters: ['default'],
    
    // 📝 序列化配置（针对 Vue 组件）
    sequence: {
      shuffle: false
    },
    
    // 🔧 设置文件
    setupFiles: [
      resolve(__dirname, 'tests/setup/global-mocks.ts')
    ],
    
    // 🚀 性能配置
    slowTestThreshold: 1000,
    
    // 🎭 UI 模式配置（Vitest 4 新位置）
    ui: false
  },

  // 🔗 解析配置
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@tests': resolve(__dirname, 'tests'),
      '@composables': resolve(__dirname, 'src/composables'),
      '@components': resolve(__dirname, 'src/components')
    }
  },

  // 📦 构建配置
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      name: 'YourAnimationLib',
      fileName: (format) => `your-animation-lib.${format}.js`
    },
    rollupOptions: {
      external: ['vue', 'motion-v'],
      output: {
        globals: {
          vue: 'Vue',
          'motion-v': 'MotionV'
        }
      }
    }
  }
})
```

### 2. **浏览器测试配置文件**：`vitest.config.browser.ts`
专门用于浏览器环境测试（Vitest 4 有重大更新）。

```typescript
import { defineConfig } from 'vitest/config'
import Vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    Vue()
  ],

  test: {
    // 🌐 浏览器模式配置（Vitest 4 新语法）
    browser: {
      enabled: true,
      // 必须指定 provider
      provider: 'playwright',
      
      // Provider 具体配置
      providers: {
        playwright: {
          launch: {
            browserName: 'chromium',
            headless: true,
            args: [
              '--no-sandbox',
              '--disable-setuid-sandbox',
              '--disable-web-security'
            ]
          }
        }
      },
      
      // 浏览器实例名称（Vitest 4 必须）
      name: 'chromium',
      
      // UI 和显示配置
      ui: false,
      headless: true
    },
    
    // 📁 文件匹配
    include: [
      'tests/component-browser/**/*.{test,spec}.ts',
      'tests/visual/**/*.{test,spec}.ts'
    ],
    
    exclude: [
      'tests/unit/**',
      'tests/composables/**',
      'tests/component/**',
      '**/node_modules/**',
      '**/dist/**'
    ],
    
    // 🎯 环境配置
    environment: 'node',
    
    // ⏱️ 超时配置（浏览器测试需要更长时间）
    testTimeout: 30000,
    hookTimeout: 30000,
    
    // 📊 覆盖率（浏览器测试中通常禁用）
    coverage: {
      enabled: false
    },
    
    // 🧪 模拟配置
    mockReset: true,
    clearMocks: true,
    
    // 🏃 运行配置
    pool: 'threads',
    poolOptions: {
      threads: {
        minThreads: 1,
        maxThreads: 2,  // 浏览器测试限制并发数
        isolate: true
      }
    },
    
    // 📈 报告配置
    reporters: ['default'],
    outputFile: {
      json: 'test-results/browser-results.json'
    },
    
    // 🔧 设置文件
    setupFiles: [
      resolve(__dirname, 'tests/setup/browser-mocks.ts')
    ],
    
    // 📝 序列化和快照
    sequence: {
      shuffle: false
    },
    
    // 🚀 性能指标
    slowTestThreshold: 5000,
    
    // 📸 快照配置
    snapshotFormat: {
      printBasicPrototype: false
    }
  },

  // 🔗 解析配置
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@tests': resolve(__dirname, 'tests'),
      '@composables': resolve(__dirname, 'src/composables'),
      '@components': resolve(__dirname, 'src/components')
    }
  },

  // 🌐 服务器配置
  server: {
    port: 5173
  }
})
```

## 🎯 必备的配套文件

### 1. **全局模拟文件**：`tests/setup/global-mocks.ts`
```typescript
import { vi } from 'vitest'

// Mock requestAnimationFrame 用于动画测试
global.requestAnimationFrame = vi.fn((callback) => {
  return setTimeout(() => callback(performance.now()), 16) as any
})

global.cancelAnimationFrame = vi.fn((id) => {
  clearTimeout(id)
})

// Mock performance.now() 用于时间控制
const originalPerformanceNow = performance.now
let mockTime = 0

Object.defineProperty(performance, 'now', {
  writable: true,
  value: vi.fn(() => mockTime)
})

// 时间控制工具（动画测试核心）
export const advanceAnimationFrames = async (frames = 1) => {
  for (let i = 0; i < frames; i++) {
    mockTime += 16
    // 执行所有 pending 的 rAF 回调
    await new Promise(resolve => setTimeout(resolve, 0))
  }
}

// Mock getComputedStyle（部分动画库需要）
global.getComputedStyle = vi.fn((element) => {
  const style = {
    opacity: '1',
    transform: 'none',
    transition: '',
    animation: ''
  }
  return style as CSSStyleDeclaration
})

// 重置时间
afterEach(() => {
  mockTime = 0
  vi.clearAllMocks()
})
```

### 2. **浏览器专用模拟文件**：`tests/setup/browser-mocks.ts`
```typescript
import { vi } from 'vitest'

// 浏览器环境中可选的额外模拟
if (typeof window !== 'undefined') {
  // Mock IntersectionObserver（用于进入视口动画）
  global.IntersectionObserver = vi.fn(() => ({
    observe: vi.fn(),
    unobserve: vi.fn(),
    disconnect: vi.fn(),
    takeRecords: vi.fn(() => [])
  })) as any

  // Mock ResizeObserver
  global.ResizeObserver = vi.fn(() => ({
    observe: vi.fn(),
    unobserve: vi.fn(),
    disconnect: vi.fn()
  })) as any
}

// 动画测试的实用函数
export const waitForAnimationFrame = () => {
  return new Promise(resolve => requestAnimationFrame(resolve))
}

export const waitForTransitionEnd = (element: HTMLElement, timeout = 1000) => {
  return new Promise<void>((resolve, reject) => {
    const timer = setTimeout(() => {
      element.removeEventListener('transitionend', onEnd)
      reject(new Error('Transition end timeout'))
    }, timeout)

    const onEnd = () => {
      clearTimeout(timer)
      resolve()
    }

    element.addEventListener('transitionend', onEnd, { once: true })
  })
}
```

## 📦 `package.json` 关键依赖配置

```json
{
  "devDependencies": {
    // 🔧 核心依赖
    "@vitejs/plugin-vue": "^5.0.0",
    "vue": "^3.4.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    
    // 🧪 Vitest 4 全套
    "vitest": "^4.0.0",
    "@vitest/ui": "^1.0.0",  // Vitest 4 中 UI 包名恢复
    "@vitest/browser": "^1.0.0",  // 浏览器模式独立包
    
    // 🌐 测试环境
    "happy-dom": "^13.0.0",
    
    // 🎭 浏览器驱动（必须）
    "@playwright/test": "^1.40.0",
    "playwright": "^1.40.0",
    
    // 📊 覆盖率
    "@vitest/coverage-v8": "^1.0.0",
    
    // 🔧 Vue 测试工具
    "@vue/test-utils": "^2.4.0",
    "vue-tsc": "^1.8.0"
  },
  "scripts": {
    // 🧪 测试脚本（Vitest 4 语法）
    "test": "vitest run",
    "test:unit": "vitest run tests/unit tests/composables",
    "test:component": "vitest run tests/component",
    "test:watch": "vitest",
    
    // 🌐 浏览器测试（必须指定配置）
    "test:browser": "vitest run --config vitest.config.browser.ts",
    "test:browser:ui": "vitest --config vitest.config.browser.ts --ui",
    "test:browser:headed": "vitest run --config vitest.config.browser.ts --browser.headless=false",
    
    // 🎭 UI 模式（Vitest 4）
    "test:ui": "vitest --ui",
    
    // 📊 覆盖率
    "test:coverage": "vitest run --coverage",
    
    // 🚀 组合命令
    "test:all": "npm run test:unit && npm run test:component && npm run test:browser",
    "test:ci": "npm run test:all -- --reporter=junit --outputFile=test-results/results.xml",
    
    // 🔄 安装浏览器
    "test:install-browsers": "playwright install chromium"
  }
}
```

## 🗂️ 推荐的测试文件结构

```
tests/
├── setup/
│   ├── global-mocks.ts        # 通用模拟
│   └── browser-mocks.ts       # 浏览器专用模拟
├── unit/
│   └── utils/
│       ├── easing.test.ts     # 缓动函数测试
│       └── physics.test.ts    # 物理动画测试
├── composables/
│   ├── useSpring.test.ts      # Spring 动画测试
│   ├── useTimeline.test.ts    # 时间轴测试
│   └── useTransition.test.ts  # 过渡动画测试
├── component/
│   ├── FadeBox.test.ts        # 渐隐组件测试（逻辑）
│   ├── SlideMenu.test.ts      # 滑动菜单测试
│   └── StaggerList.test.ts    # 交错列表测试
└── component-browser/
    ├── FadeBox.render.test.ts # 渲染测试
    ├── SlideMenu.visual.test.ts # 视觉测试
    └── StaggerList.animation.test.ts # 动画时序测试
```

## 🔧 Vitest 4 关键变化和适配说明

### 1. **浏览器模式配置变化**
```typescript
// ❌ Vitest 3 及之前
browser: {
  enabled: true,
  provider: 'playwright'
}

// ✅ Vitest 4 新语法
browser: {
  enabled: true,
  provider: 'playwright',  // 必须指定
  providers: {
    playwright: {
      launch: {
        browserName: 'chromium',
        headless: true
      }
    }
  },
  name: 'chromium'  // 必须指定实例名称
}
```

### 2. **安装注意事项**
```bash
# 必须安装的包
npm install -D vitest@4 @vitest/browser @playwright/test playwright

# 安装 Playwright 浏览器
npx playwright install chromium

# 如果使用 UI 模式
npm install -D @vitest/ui
```

### 3. **运行命令差异**
```bash
# Vitest 4 浏览器测试必须指定配置文件
vitest run --config vitest.config.browser.ts

# 或者通过脚本简化
npm run test:browser
```

### 4. **动画测试的特殊处理**
在动画库测试中，强烈建议使用时间控制：
```typescript
// 在测试文件中
import { vi } from 'vitest'

test('spring 动画应该在指定时间内完成', async () => {
  vi.useFakeTimers()
  
  const { result } = renderHook(() => useSpring({ target: 100, duration: 1000 }))
  result.current.start()
  
  // 快进时间
  await vi.advanceTimersByTimeAsync(500)
  expect(result.current.value).toBeCloseTo(50, -1)
  
  await vi.advanceTimersByTimeAsync(500)
  expect(result.current.value).toBe(100)
  
  vi.useRealTimers()
})
```

## 🚀 开始使用步骤

1. **安装依赖**：
   ```bash
   npm install
   npm run test:install-browsers
   ```

2. **运行基础测试**：
   ```bash
   # 逻辑测试（快速反馈）
   npm run test:component
   
   # 浏览器渲染测试
   npm run test:browser
   ```

3. **CI/CD 配置**：
   ```bash
   # 典型 CI 脚本
   npm ci
   npm run test:install-browsers
   npm run test:ci
   ```

这个配置方案完全基于 Vitest 4.0.0，确保了最佳的兼容性和性能。对于动画库测试，特别优化了时间控制和浏览器渲染验证。如果你在配置过程中遇到任何问题，可以告诉我具体的错误信息，我会帮你调整配置。