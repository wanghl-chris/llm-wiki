---
title: "HOS 平台前端工程迁移与 Bug 修复完整指南"
type: article
created: 2026-04-23
keywords: [HOS 平台, 前端迁移, 工程迁移, Git 分支, ECharts, 路由配置]
---

# hos 分支迁移执行计划

## 概述
本文档记录了将 `frontend-cdc-aidp` 工程在 hos 分支上的最后一次提交改动迁移至 `frontend-cdc-afsim` 工程的详细步骤和注意事项，以及后续的 bug 修复内容，以便后续迁移其他工程时直接复用。

### 最新更新：hos集成bug修复
**提交信息**：hos集成bug修复
**提交时间**：2026-04-23 21:24:05
**提交哈希**：991302499d6289c062844feeea69e640764ce2bb
**涉及文件**：
- package-lock.json
- package.json
- src/components/Echarts/unsed-options/Base.ts
- src/components/Echarts/uses/useInitBarEcharts.ts
- src/components/Echarts/uses/useInitEcharts.ts
- src/routers/index.ts

**修复内容**：
1. **ECharts 类型定义修复**：
   - 将 `EChartsOption` 改为 `EChartOption`，解决类型定义不匹配问题
   - 涉及文件：`src/components/Echarts/unsed-options/Base.ts`、`src/components/Echarts/uses/useInitBarEcharts.ts`、`src/components/Echarts/uses/useInitEcharts.ts`

2. **路由配置优化**：
   - 添加路由重复跳转的处理逻辑，避免控制台报错
   - 移除不必要的导入（`removeSessionCache`、`RequestUtil`）
   - 调整 `getMenuList` 函数，使用 `generateAuthRoutes` 生成路由
   - 移除硬编码的区域设置（注释掉 `_utils.setSessionCache('realArea', ['420000'])` 等相关代码）
   - 优化导入语句，将 `getHosSessionInfo` 单独导入
   - 移除不必要的控制台日志和注释

3. **代码格式调整**：
   - 移除不必要的控制台日志（如 `console.time` 和 `console.timeEnd`）
   - 调整代码缩进和格式（如 `console.log` 语句的空格）
   - 调整对象定义格式（如 `const combinedRoute: any = {`）
   - 移除尾部分号，保持代码风格一致

4. **依赖更新**：更新了 package.json 和 package-lock.json，确保依赖版本与 hos 环境兼容

**解决思路**：
- 针对 ECharts 类型定义问题，统一使用正确的类型名称 `EChartOption`，确保与 echarts 4.9.0 版本兼容
- 针对路由配置问题，优化了路由生成逻辑，使用 hos 提供的 `generateAuthRoutes` 工具，提高路由生成的可靠性和一致性
- 针对路由重复跳转问题，通过重写 `VueRouter.prototype.push` 方法，捕获并处理路由重复跳转的错误
- 针对代码质量问题，移除了不必要的代码和日志，提高代码可读性和执行效率
- 针对硬编码问题，移除了硬编码的区域设置，改为从 hos 会话信息中动态获取

**对整体迁移计划的影响**：
- 确保了 ECharts 图表在 hos 环境下的正常显示，避免了类型定义错误
- 优化了路由生成逻辑，提高了菜单加载的可靠性和一致性
- 解决了路由重复跳转的控制台报错问题，提升了用户体验
- 减少了代码冗余，提高了代码质量和可维护性
- 为后续其他工程的迁移提供了更完整的参考，特别是 ECharts 类型定义和路由配置方面
- 强调了代码格式的统一性，为团队协作提供了更好的规范

## 迁移前准备

### 1. 获取源工程的提交记录
```bash
cd /path/to/frontend-cdc-aidp
git checkout hos
git log -1 --name-status
git show <commit-hash> --name-only
```

### 2. 分析改动内容
重点检查以下类型的改动：
- 配置文件（.env、package.json、vite.config.js）
- 核心业务逻辑文件（HttpRequest.ts、routers/index.ts、main.ts）
- 组件文件（Layout.vue 等）
- 公共工具文件（utils、apis 等）

## 详细迁移步骤

### 第一步：环境配置文件迁移

#### 1.1 修改 .env 文件
**位置**：项目根目录
**改动内容**：
- 更新 `VUE_APP_LOCAL_STATIC_PATH` 从 public 地址改为内网地址
- 更新 `VUE_APP_BASE_PROXY_URL` 为内网地址
- 更新 `VUE_APP_BASE_DISEASE_CONTROL` 为内网地址
- 确保其他环境变量与源工程保持一致

**示例**：
```env
# 本地调试时的代理地址
VUE_APP_BASE_PROXY_URL=http://172.19.19.25:9005

#静态资源的路径
VUE_APP_STATIC_PATH=/static/
VUE_APP_LOCAL_STATIC_PATH=http://172.19.19.25:9005/static/
VUE_APP_BASE_OSS_UPLOAD=/
VUE_APP_BASE_DISEASE_CONTROL=http://172.19.19.25:9005
```

#### 1.2 修改 package.json 文件
**位置**：项目根目录
**改动内容**：

**scripts 部分**：
```json
{
  "scripts": {
    "install-package": "npm i && npm i --registry=https://packages.aliyun.com/6463532797d94d909e43a854/npm/npm-registry/ medical-core@^3.0.2 medical-component@^3.0.2 medical-bs-component@^1.0.1",
    "vite": "vite --host",
    "devgo": "npm run vite",
    "prodgo": "cross-env VUE_APP_BASE_PROXY_URL=http://88.7.129.120:50001 npm run vite"
  }
}
```

**dependencies 部分**：
- 移除：`echarts`、`echarts-wordcloud`、`el-table-infinite-scroll`、`moment`
- 添加/更新：
  - `medical-bs-component@^1.0.1`
  - `medical-component@^3.0.2`
  - `medical-core@^3.0.11`（注意：根据私有注册表中可用的版本进行调整）
  - `vue-echarts@^4.1.0`
  - `echarts@4.9.0`（重要：使用 4.9.0 而非 5.x 版本）
  - `echarts-wordcloud@^1.1.3`

**devDependencies 部分**：
- 添加 Vite 相关依赖：
  - `@vitejs/plugin-vue2@2.3.1`
  - `vite@5.0.12`
  - `@originjs/vite-plugin-commonjs@^1.0.3`
  - `@originjs/vite-plugin-require-context@^1.0.9`
  - `vite-plugin-html@^3.2.2`
  - `vite-plugin-commonjs@0.10.1`
  - `vite-plugin-node-polyfills@0.19.0`
  - `vite-plugin-require@1.1.14`
  - `vite-plugin-svg-icons@2.0.1`
- 添加 importus 插件所需依赖：
  - `acorn@^8.11.3`
  - `es-module-lexer@^1.4.1`
  - `lodash@^4.17.21`
  - `magic-string@^0.30.5`
  - `@types/lodash@^4.14.202`

**修改要点**：
1. install-package 脚本需要使用私有注册表安装 medical-* 相关依赖
2. vite 命令需要添加 --host 参数，以便在局域网内访问
3. echarts 必须使用 4.9.0 版本，不能使用 5.x 版本，否则会出现导入错误
4. medical-core 版本需要根据私有注册表中可用的版本进行调整
5. importus 插件所需的依赖必须全部安装，否则会导致按需引入失败

### 第二步：Vite 配置文件迁移

#### 2.1 创建 vite.config.js 文件
**位置**：项目根目录
**内容**：从源工程复制完整的 vite.config.js 文件

**关键配置项**：
- 插件配置：vue2、viteCommonjs、vitePluginImportus、viteRequireContext、svgBuilder、createHtmlPlugin
- 代理配置：/api、/test-proxy
- 别名配置：@/、~@/
- 环境变量配置：process.env

#### 2.2 创建 index.vite.html 文件
**位置**：项目根目录
**内容**：从源工程复制完整的 index.vite.html 文件

#### 2.3 创建 public/importus.ts 文件
**位置**：public 目录
**内容**：从源工程复制完整的 importus.ts 文件

**重要说明**：
- 使用 acorn 解析器和 MagicString 处理导入语句
- 不要使用正则表达式方式实现（会导致 Vite 构建错误）

#### 2.4 创建 public/svgBuilder.ts 文件
**位置**：public 目录
**内容**：从源工程复制完整的 svgBuilder.ts 文件

### 第三步：核心业务逻辑迁移

#### 3.1 修改 src/common/HttpRequest.ts
**改动内容**：
- 修改导入语句：
  ```typescript
  // 从
  import { requestUtil } from 'medical-core/utils/RequestUtil'
  // 改为
  import { requestUtil } from 'medical-core/utils/HosRequestUtil'
  ```

- 注释掉 token 移除代码：
  ```typescript
  // requestUtil.removeToken(Cons.TOKEN_ID) // 注释掉这一行
  ```

- 调整代码格式：
  - 移除尾部分号
  - 简化 baseRequest 的 then 回调逻辑

**详细修改说明**：
1. 原有的 then 回调逻辑包含多层嵌套的条件判断，需要简化
2. 新的逻辑将所有成功情况合并到一个条件中
3. 保持会话超时处理逻辑不变

**修改前的 then 回调逻辑**：
```typescript
then(response => {
  const status = response.status
  const data = response.data
  if (status === 200) {
    if (data.success || data.code == '000000') {
      resolve(data)
    } else {
      if (request_.url.includes('/letter/get-unread')) {
        // 获取站内信接口，在登录过期时不跳转登录页
        // resolve(data);
      } else if (data.errorCode == 'SYS2002' || data.errorCode == 'SYS2007' || data.errorCode == '10001') {
        // 如果是内嵌页面 超时之后让主页面同步退出登录
        console.log('会话超时 退出登录')
        if (window.parent !== window.self) {
          window.parent.postMessage('reLogin', window.location.origin)
        }
        // 判断是否登录过期
        reLogin(data.errorMsg)
        return
      }
      // 字节流直接输出
      if (isBlob(data)) return resolve(data)

      if (requestParameters.setting.isShowErr !== false) {
        // ScMessage.error(data && (data.retMsg || data.errorMsg))
        console.error(data && (data.retMsg || data.errorMsg || data.errorCode))
      } else if (bobTypeRequestList.some(element => request_.url.includes(element))) {
        // 传染病直报监测详情页面 -- 图片显示(接口返回的是数据流，非一般response格式)
        return resolve(data)
      }
      if (data.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' || data.type === 'application/msexcel' || data.type == 'application/vnd.ms-excel' || data.type == 'application/octet-stream') {
        resolve(data)
        return
      }
      reject(data)
    }
  } else {
    if (requestParameters.setting.isShowErr !== false) {
      // ScMessage.error(data && (data.retMsg || data.errorMsg || ''))
      console.error(data && (data.retMsg || data.errorMsg || data.errorCode))
    }
    reject(response)
  }
})
```

**修改后的 then 回调逻辑**：
```typescript
then(response => {
  if (response.success || response.code == '000000' || isBlob(response) || bobTypeRequestList.some(item => request_.url.includes(item))) {
    return resolve(response)
  } else {
    if (response.errorCode == 'SYS2002' || response.errorCode == 'SYS2007' || response.errorCode == '10001') {
      // 如果是内嵌页面 超时之后让主页面同步退出登录
      console.log('会话超时 退出登录')
      if (window.parent !== window.self) {
        window.parent.postMessage('reLogin', window.location.origin)
      }
      // 判断是否登录过期
      reLogin(response.errorMsg)
      return
    }
    if (requestParameters.setting.isShowErr !== false) {
      // ScMessage.error(response && (response.retMsg || response.errorMsg || ''))
      console.error(response && (response.retMsg || response.errorMsg || response.errorCode))
    }
    return reject(response)
  }
})
```

**修改要点**：
1. 移除了对 response.status 的判断，直接使用 response 对象
2. 将所有成功情况（success、code为000000、blob数据、特殊接口）合并到一个条件中
3. 简化了错误处理逻辑，移除了冗余的条件判断
4. 保持了会话超时处理逻辑不变

#### 3.2 修改 src/routers/index.ts
**改动内容**：

1. 添加 hos 相关函数导入：
   ```typescript
   import { getHosAuthMenu, getHosSessionInfo } from '@/apis/Uc'
   import { generateAuthRoutes } from 'medical-core/utils/hosAuthRoutes'
   ```

2. 添加路由重复跳转处理逻辑：
   ```typescript
   // 处理重复跳转报错提示
   const originalPush = VueRouter.prototype.push
   VueRouter.prototype.push = function push(location) {
          return originalPush.call(this, location).catch(err => err)
   }
   ```

3. 修改 getMenuList 函数：
   ```typescript
   // 从
   export async function getMenuList(callback?) {
     if (window.dynamicRouter) {
       callback && callback(window.dynamicRouter)
       return window.dynamicRouter
     }
     const response = await ucGetMenuList()
     // ... 其他代码
   }
   
   // 改为
   export async function getMenuList(callback?) {
     if (window.dynamicRouter) {
       callback && callback(window.dynamicRouter)
       return window.dynamicRouter
     }
     const response = await getHosAuthMenu({
       resourceCode: 'AIDP', // 根据实际项目修改资源代码
     })
     if (response.data && response.data.menus && response.data.menus[0] && response.data.menus[0].children) {
       _utils.setSessionCache('menuList', response.data.menus[0].children)
       let list = response.data.menus[0].children
       window.originMenuList = list
       if (list && list.length > 0) {
         let defaultUrl = ''
         // 递归获取 首个叶子节点
         const mapReducer = mrdlist => {
           defaultUrl += (mrdlist[0] || '') && mrdlist[0].path
           if (mrdlist[0] && mrdlist[0].children) {
             mapReducer(mrdlist[0].children)
           } else {
             return defaultUrl
           }
         }
         const _routerList = generateAuthRoutes({
           asyncRouter: asyncRouterMap, // 原始动态路由表
           authMenu: list,
         })
         mapReducer(_routerList)
         defaultUrl = sessionStorage.getItem('isElsaBS') ? '/personalLocus' : defaultUrl
         defaultUrl = defaultUrl || '/404'
         window.dynamicRouter = _routerList
         window.defaultUrl = defaultUrl
         callback && callback(_routerList)
         return _routerList
       }
     }
   }
   ```

4. 修改 loadTenantInfo 函数：
   ```typescript
   // 从
   async function loadTenantInfo() {
     if (stores.state.currentTenantArea.currentCodeList.length !== 0) return
     const res = await societyGetUserInfo(tenantObject)
     // ... 其他代码
   }
   
   // 改为
   async function loadTenantInfo() {
     if (stores.state.currentTenantArea.currentCodeList.length !== 0) return
     const res = await getHosSessionInfo()
     setUserInfo({
       nickName: res.data.nickName,
       headerIcon: res.data.headerIcon,
       tenantName: res.data.orgBaseInfo.tenantName,
     })
     getCurrentAccountMapArea(res.data)
     stores.commit(storeTypes.CITY_LEVEL, res.data.orgBaseInfo.divisionType)
     const tenantAreaInfo = {
       name: res.data.orgBaseInfo.provinceName,
       currentNameList: [],
       currentCodeList: [],
     }
     if (res.data.orgBaseInfo.divisionType == '1') {
       tenantAreaInfo.currentNameList = [res.data.orgBaseInfo.provinceName.trim()]
       tenantAreaInfo.currentCodeList = [res.data.orgBaseInfo.provinceCode.slice(0, 6)]
     } else if (res.data.orgBaseInfo.divisionType == '2') {
       tenantAreaInfo.currentNameList = [res.data.orgBaseInfo.provinceName.trim(), res.data.orgBaseInfo.cityName.trim()]
       tenantAreaInfo.currentCodeList = [res.data.orgBaseInfo.provinceCode.slice(0, 6), res.data.orgBaseInfo.cityCode.slice(0, 6)]
     } else {
       tenantAreaInfo.currentNameList = [res.data.orgBaseInfo.provinceName.trim(), res.data.orgBaseInfo.cityName.trim(), res.data.orgBaseInfo.districtName.trim()]
       tenantAreaInfo.currentCodeList = [res.data.orgBaseInfo.provinceCode.slice(0, 6), res.data.orgBaseInfo.cityCode.slice(0, 6), res.data.orgBaseInfo.districtCode.slice(0, 6)]
     }
     // 在当前选中地区权限范围大于租户区域时，用租户地区更新当前地区
     if (stores.state.currentHeaderArea.currentNameList.length < tenantAreaInfo.currentNameList.length) {
       stores.commit(storeTypes.MUTATION_CURRENT_HEADER_AREA, tenantAreaInfo)
     }
     sessionStorage.setItem('realArea', JSON.stringify(tenantAreaInfo.currentCodeList))
     sessionStorage.setItem('selectedArea', JSON.stringify(tenantAreaInfo.currentCodeList))
     sessionStorage.setItem('selectAreaName', tenantAreaInfo.name)
     stores.commit(storeTypes.MUTATION_CURRENT_TENANT_AREA, tenantAreaInfo)
   }
   ```

5. 移除硬编码的区域设置：
   ```typescript
   // 从
   _utils.setSessionCache('realArea', ['420000'])
   _utils.setSessionCache('selectedArea', ['420000'])
   _utils.setSessionCache('selectAreaName', '湖北省')
   
   // 改为
   // _utils.setSessionCache('realArea', ['420000'])
   // _utils.setSessionCache('selectedArea', ['420000'])
   // _utils.setSessionCache('selectAreaName', '湖北省')
   ```

6. 移除不必要的控制台日志和注释：
   ```typescript
   // 移除
   console.time('预请求')
   // ... 代码
   console.timeEnd('预请求')
   
   // 移除
   // 金山云免密登录
   
   // 简化
   console.log('dynamicRouterStore.hasAdded', dynamicRouterStore.hasAdded)
   ```

**修改要点**：
1. 添加 hos 相关函数导入，包括 getHosAuthMenu、getHosSessionInfo 和 generateAuthRoutes
2. 添加路由重复跳转处理逻辑，避免控制台报错
3. 修改 getMenuList 函数，使用 getHosAuthMenu 替代 ucGetMenuList，并使用 generateAuthRoutes 生成路由
4. 修改 loadTenantInfo 函数，使用 getHosSessionInfo 替代 societyGetUserInfo，并调整数据处理逻辑
5. 移除硬编码的区域设置，改为从 hos 会话信息中动态获取
6. 移除不必要的控制台日志和注释，提高代码可读性
7. 注意 resourceCode 需要根据实际项目进行修改

#### 3.3 修改 src/main.ts
**改动内容**：
- 添加 hos 样式导入：
  ```typescript
  import 'medical-core/style/hos-icon.css'
  ```

#### 3.4 修改 src/apis/Uc.ts
**改动内容**：
- 添加 hos 相关函数：
  ```typescript
  export function getHosAuthMenu(params) {
    return request({
      url: `/sys/resources/list-menu`,
      method: 'post',
      params,
    })
  }

  export function getHosSessionInfo() {
    return request({
      url: `/phes/base/cdc/session/info`,
      method: 'post',
      params: {},
    })
  }
  ```

**修改要点**：
1. getHosAuthMenu 函数用于获取 hos 认证菜单，需要传入 resourceCode 参数
2. getHosSessionInfo 函数用于获取 hos 会话信息，包括用户信息和租户信息
3. 这两个函数使用 request 方法进行请求，与项目中的其他 API 函数保持一致

#### 3.5 修改 ECharts 类型定义
**改动内容**：
- 修改 `src/components/Echarts/unsed-options/Base.ts`：
  ```typescript
  // 从
  const options: EChartsOption = {
  // 改为
  const options: EChartOption = {
  ```

- 修改 `src/components/Echarts/uses/useInitBarEcharts.ts`：
  ```typescript
  // 从
  (event: 'before', instance: echarts.ECharts, options: echarts.EChartsOption): void;
  // 改为
  (event: 'before', instance: echarts.ECharts, options: echarts.EChartOption): void;
  ```

- 修改 `src/components/Echarts/uses/useInitEcharts.ts`：
  ```typescript
  // 从
  (event: 'before', instance: echarts.ECharts, options: echarts.EChartsOption): void;
  // 改为
  (event: 'before', instance: echarts.ECharts, options: echarts.EChartOption): void;
  ```

**修改要点**：
1. 将所有 `EChartsOption` 改为 `EChartOption`，确保与 echarts 4.9.0 版本的类型定义兼容
2. 涉及所有使用 ECharts 类型的文件，确保类型定义的一致性
3. 这是解决 ECharts 导入错误的关键步骤

### 第四步：组件文件迁移

#### 4.1 修改 src/components/layout/Layout.vue
**改动内容**：

1. 添加 hasScrollBarTimer 属性：
   ```typescript
   public hasScrollBarTimer = null
   ```

2. 修改 userInfo 初始化值：
   ```typescript
   // 从
   public userInfo: any = this.$userUtil.getUserInfo()
   // 改为
   public userInfo: any = {}
   ```

3. 修改 nickUserIcon getter 方法，强制使用默认头像：
   ```typescript
   get nickUserIcon(): string {
     //this.useSrcUserIcon = !this.userInfo.headerIcon
     this.useSrcUserIcon = true
     return require('@/assets/common/icon_defaulthead.png')
   }
   ```

4. 修改 created 方法：
   ```typescript
   // 从
   if (this.userInfo && this.$utils.getSessionCache('token')) {
     // 有登录的情况下
     this.routerMap = [...constantRouterMap, ...asyncRouterMap]
     // router.addRoutes([...constantRouterMap, ...asyncRouterMap])
     setTimeout(() => {
       this.isSecuritySettingAllow()
     }, 1000)
     getMenuList(this.getMenuCallBack)
   }
   
   // 改为
   getMenuList(this.getMenuCallBack)
   // setTimeout(() => {
   //   this.isSecuritySettingAllow()
   // }, 1000)
   if (this.userInfo && this.$utils.getSessionCache('token')) {
     // 有登录的情况下
     this.routerMap = [...constantRouterMap, ...asyncRouterMap]
     // router.addRoutes([...constantRouterMap, ...asyncRouterMap])
   }
   ```

5. 修改 getMenuCallBack 方法，添加用户信息获取：
   ```typescript
   public getMenuCallBack(map) {
     this.routerMap = map
     this.getMenusFlag = true
     this.getBreadcrumbList(this.$route)
     this.userInfo = this.$userUtil.getUserInfo()
   }
   ```

6. 调整 handle$route 方法中的滚动条相关代码格式：
   ```typescript
   clearTimeout(this.hasScrollBarTimer)
   this.hasScrollBarTimer = setTimeout(() => {
     const el = document.querySelector('.main-container .app-main')
     if (!el) {
       return
     }
     if (el.scrollHeight > el.clientHeight) {
       el.classList.add('has-scroll-bar')
     } else {
       el.classList.remove('has-scroll-bar')
     }
   }, 500)
   ```

7. 修改 goToRouter 方法中的 nameArr 数组格式，调整为单行：
   ```typescript
   public goToRouter(item, index, length) {
     const nameArr = ['CSM_VISIT_DETAIL_A_IN', 'CSM_VISIT_DETAIL_A_OU', 'EW_SUFFERER_HOSPITALIZE_DETAIL', 'SUFFERER_HOSPITALIZE_DETAIL_FROM_INFC_DETAIL', 'SUFFERER_HOSPITALIZE_DETAIL_FROM_INFC', 'CSM_SUFFERER_DETAIL_FROM_FEVERILISARI', 'CSM_VISIT_DETAIL_C', 'CSM_SYNDROME_AGG_VISITDETAIL', 'CSM_SYNDROME_CASE_VISITDETAIL', 'EW_VISIT_DETAIL_B', 'EW_VISIT_DETAIL_ANALYZE', 'DIAGNOSTIC_REMINDERS_DETAIL']
     
     console.log(111111, item, index)
     if (nameArr.some(it => item.name.includes(it))) {
       // 解决医疗机构点击就诊详情URL参数消失问题
       return this.$router.go(-1)
     }
     
     let { name } = item
     if (item.redirect) {
       name = item.redirect
     }
     // 最后一级面包屑不用跳转，jumpable属性为false时不可跳转
     if ((item.jumpable == undefined || item.jumpable == true) && index !== this.breadList.length - 1) {
       this.$router.push({ name })
     }
   }
   ```

8. 调整 logout 方法中的代码格式，移除分号：
   ```typescript
   public logout(): void {
     const isFromPortal = JSON.parse(window.sessionStorage.getItem('isFromPortal'))
     if (window.opener) {
       window.opener = null
       window._open('', '_self')
       window.close()
     } else {
       Api.uc.logout().finally(() => {
         // handleLogout()
         requestUtil.removeToken(window.tokenId)
         window.sessionStorage.clear()
         // window.location.reload() // 刷新页面 清空路由、store里面的数据
         if (process.env.NODE_ENV === 'production') {
           location.href = '/portal/login?lastLogoutPage=' + btoa(window.location.href)
         } else {
           this.$router.push({
             path: '/login',
             query: {
               lastLogoutPage: btoa(window.location.href),
             },
           })
         }
       })
     }
   }
   ```

**修改要点**：
1. 添加 hasScrollBarTimer 属性用于控制滚动条显示的定时器
2. 将 userInfo 初始化为空对象，避免在组件创建时获取用户信息
3. 强制使用默认头像，不再根据用户信息动态设置
4. 将 getMenuList 调用移到条件判断之外，确保菜单始终加载
5. 在 getMenuCallBack 中获取用户信息，确保用户信息在菜单加载后获取
6. 调整代码格式，移除尾部分号，保持代码风格一致
7. 将 nameArr 数组调整为单行格式，提高代码可读性

### 第五步：依赖安装和验证

#### 5.1 安装依赖
```bash
npm install
```

#### 5.2 启动开发服务器
```bash
npm run vite
```

#### 5.3 验证关键功能
- 检查浏览器控制台是否有错误
- 验证菜单加载功能
- 验证租户信息获取功能
- 验证 ECharts 图表显示
- 验证登录登出功能

## 常见问题和解决方案

### 问题 1：Vite 构建错误 - 正则表达式语法错误
**错误信息**：
```
Invalid regular expression: /imports+((?:*s+ass+)?w+)s+froms+['"]medical-component['"]/: Nothing to repeat
```

**解决方案**：
- 确保 public/importus.ts 文件使用 acorn 解析器和 MagicString 实现
- 不要使用正则表达式方式实现

### 问题 2：ECharts 导入错误
**错误信息**：
```
Uncaught SyntaxError: The requested module '/node_modules/.vite/deps/echarts.js?v=2f203457' does not provide an export named 'default'
```

**解决方案**：
- 将 echarts 版本降级到 4.9.0
- 将 echarts-wordcloud 版本降级到 1.1.3
- 确保 package.json 中的依赖版本与 medical-component 兼容

### 问题 3：medical-component 按需引入失败
**解决方案**：
- 确保 vite.config.js 中正确配置了 vitePluginImportus 插件
- 确保 public/importus.ts 文件存在且内容正确
- 确保 acorn、es-module-lexer、magic-string、lodash 依赖已安装

### 问题 4：SVG 图标不显示
**解决方案**：
- 确保 public/svgBuilder.ts 文件存在且内容正确
- 确保 src/assets/icons/ 目录下有 SVG 文件
- 确保 vite.config.js 中正确配置了 svgBuilder 插件

### 问题 5：私有注册表依赖安装失败
**错误信息**：
```
npm ERR! code ETARGET
npm ERR! notarget No matching version found for medical-core@^3.1.15.
```

**解决方案**：
- 检查私有注册表中可用的 medical-core 版本
- 根据实际可用版本调整 package.json 中的版本号
- 使用私有注册表安装依赖：`npm install --registry=https://packages.aliyun.com/6463532797d94d909e43a854/npm/npm-registry/`

### 问题 6：Node.js 版本不兼容
**错误信息**：
```
npm WARN EBADENGINE Unsupported engine { package: 'vite@5.0.12', required: { node: '^18.0.0 || >=20.0.0' }, current: { node: 'v16.20.2', npm: '8.19.4' } }
```

**解决方案**：
- 忽略警告，Vite 5.0.12 可以在 Node.js 16.20.2 上运行
- 或者升级 Node.js 到 18.0.0 或更高版本

## 验证清单

### 配置文件验证
- [ ] .env 文件已更新
- [ ] package.json 文件已更新
- [ ] vite.config.js 文件已创建
- [ ] index.vite.html 文件已创建
- [ ] public/importus.ts 文件已创建
- [ ] public/svgBuilder.ts 文件已创建

### 核心业务逻辑验证
- [ ] HttpRequest.ts 已修改
- [ ] routers/index.ts 已修改
- [ ] main.ts 已修改
- [ ] Uc.ts 已添加 hos 相关函数

### 组件文件验证
- [ ] Layout.vue 已修改

### 功能验证
- [ ] 开发服务器可以正常启动
- [ ] 浏览器控制台无错误
- [ ] 菜单加载功能正常
- [ ] 租户信息获取功能正常
- [ ] ECharts 图表显示正常
- [ ] 登录登出功能正常

## 注意事项

1. **版本兼容性**：
   - echarts 必须使用 4.9.0 版本，不能使用 5.x 版本
   - echarts-wordcloud 必须使用 1.1.3 版本
   - medical-component、medical-core、medical-bs-component 版本必须与源工程一致
   - 确保所有 ECharts 类型定义使用 `EChartOption` 而非 `EChartsOption`

2. **代码风格**：
   - 移除尾部分号，保持代码风格一致
   - 调整数组、对象格式为单行形式
   - 移除不必要的控制台日志和注释，提高代码可读性

3. **依赖安装**：
   - 修改 package.json 后必须运行 npm install
   - 如果遇到依赖冲突，可以删除 node_modules 和 package-lock.json 后重新安装
   - 使用私有注册表安装 medical-* 相关依赖

4. **测试验证**：
   - 迁移完成后必须进行充分的功能测试
   - 特别关注菜单加载、租户信息获取、图表显示等核心功能
   - 测试路由重复跳转是否会导致控制台报错

5. **Git 提交**：
   - 迁移完成后先进行人工验证，确认修改方案的可行性及兼容性
   - 确保迁移内容不会对目标工程造成负面影响
   - 验证通过后再提交到 hos 分支

6. **路由配置**：
   - 必须添加路由重复跳转的处理逻辑，避免控制台报错
   - 使用 `generateAuthRoutes` 生成路由，确保路由配置的一致性
   - 移除硬编码的区域设置，改为从 hos 会话信息中动态获取

7. **ECharts 配置**：
   - 确保所有 ECharts 相关文件的类型定义使用 `EChartOption`
   - 检查所有使用 ECharts 的组件，确保类型定义的一致性

8. **安全注意**：
   - 确保 hos 相关的 API 调用使用正确的权限验证
   - 避免在代码中硬编码敏感信息
   - 确保会话超时处理逻辑正确实现

## 潜在风险提示

1. **ECharts 版本兼容性风险**：
   - 如果使用 echarts 5.x 版本，会出现类型定义和导入错误
   - 解决方案：严格使用 echarts 4.9.0 版本和 echarts-wordcloud 1.1.3 版本

2. **路由配置风险**：
   - 路由重复跳转可能导致控制台报错，影响用户体验
   - 硬编码的区域设置可能导致在不同环境下的显示错误
   - 解决方案：添加路由重复跳转处理逻辑，使用动态区域设置

3. **依赖安装风险**：
   - 私有注册表依赖版本不兼容可能导致安装失败
   - 解决方案：检查私有注册表中可用的版本，调整 package.json 中的版本号

4. **类型定义风险**：
   - ECharts 类型定义不一致可能导致编译错误
   - 解决方案：统一使用 `EChartOption` 类型定义

5. **API 调用风险**：
   - hos 相关 API 调用失败可能导致功能不可用
   - 解决方案：确保 API 调用参数正确，添加错误处理逻辑

6. **代码风格风险**：
   - 代码风格不一致可能导致团队协作困难
   - 解决方案：遵循统一的代码风格规范，移除尾部分号，调整格式

## 后续迁移其他工程的建议

1. **创建迁移脚本**：
   - 可以将上述步骤自动化，减少人工操作错误
   - 使用脚本自动对比和复制文件

2. **建立迁移检查清单**：
   - 使用本文档的验证清单，确保所有步骤都已完成
   - 记录每次迁移遇到的问题和解决方案

3. **版本控制**：
   - 在迁移前创建备份分支
   - 迁移过程中及时提交，便于回滚

4. **团队协作**：
   - 将本文档分享给团队成员
   - 定期更新文档，补充新的问题和解决方案

5. **统一迁移标准**：
   - 建立统一的迁移标准和规范
   - 确保所有工程的 hos 分支迁移保持一致

6. **预测试验证**：
   - 在正式迁移前，在测试环境中验证迁移方案
   - 确保迁移不会影响现有功能

7. **文档维护**：
   - 定期更新迁移文档，记录新的问题和解决方案
   - 为每个工程创建迁移日志，便于追溯和参考

## 总结

本执行计划详细记录了 hos 分支迁移的所有步骤和注意事项，涵盖了配置文件、核心业务逻辑、组件文件等各个方面的改动。通过遵循本计划，可以确保迁移过程的准确性和一致性，减少错误和遗漏。在实际迁移过程中，如遇到新的问题，请及时更新本文档，以便后续参考。