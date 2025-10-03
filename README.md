# RailGo - 现代化铁路查询工具

<div align="center">
  <img src="static/logo.png" alt="RailGo Logo" width="120" height="120">
  
  <h3>一个现代化的专业铁路查询工具</h3>
  
  [![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
  [![UniApp](https://img.shields.io/badge/UniApp-Vue3-green.svg)](https://uniapp.dcloud.io/)
  [![Platform](https://img.shields.io/badge/platform-iOS%20%7C%20Android%20%7C%20HarmonyOS-lightgrey.svg)]()
</div>

## 📖 项目简介

RailGo 是一个基于 UniApp 开发的现代化铁路查询工具，为铁路爱好者和旅客提供全面的列车信息查询服务。本项目采用 Vue 3 + UniApp 技术栈，支持多平台部署，包括 iOS、Android、鸿蒙。

**⚠️ 重要声明：** 本软件为非官方软件，与中国国家铁路集团总公司、中国铁道科学研究院集团有限公司等铁路运营企业及其下级企业单位无任何隶属关系。所有数据仅供学习参考，请以铁路运营企业实际运用为准。

## ✨ 主要功能

### 🚄 车次查询
- **车次信息查询**：支持通过车次号查询列车详细信息
- **站站查询**：查询两站间的直达列车
- **时刻表查询**：完整的列车时刻表信息

### 🏢 车站服务
- **车站信息查询**：全国铁路车站基础信息
- **站点选择**：智能站点搜索和选择

### 🚅 动车组查询
- **EMU查询**：动车组担当信息查询
- **配属信息**：动车组配属段信息
- **车型展示**：丰富的动车组车型图片库

### 📊 数据服务
- **实时数据**：支持实时数据查询
- **离线模式**：本地数据库支持离线查询
- **数据更新**：支持数据库在线更新

## 🛠️ 技术栈

### 前端技术
- **框架**：UniApp + Vue 3
- **状态管理**：Vue 3 Composition API
- **UI组件**：FirstUI + 自定义组件
- **网络请求**：Axios + UniApp适配器
- **数据存储**：SQLite + JSON数据库

### 后端服务
- **API服务**：Flask (Python)
- **数据格式**：JSON
- **跨域支持**：CORS
- **接口文档**：RESTful API

### 开发工具
- **IDE**：HBuilderX
- **版本控制**：Git
- **包管理**：npm/pnpm
- **构建工具**：UniApp CLI

## 📱 支持平台

- ✅ **Android** - 原生App
- ✅ **iOS** - 原生App  
- ✅ **鸿蒙系统** (HarmonyOS)

## 🚀 快速开始

### 环境要求

- **Node.js** >= 16.0.0
- **HBuilderX** >= 3.8.0 (推荐使用最新版)
- **Python** >= 3.8 (用于API服务器)

### 安装步骤

1. **克隆项目**
```bash
git clone https://github.com/your-username/RailGo-UniApp.git
cd RailGo-UniApp
```

2. **安装依赖**
```bash
npm install
# 或使用 pnpm
pnpm install
```

3. **启动API服务器**
```bash
cd api
pip install -r requirements.txt
python app.py
```

4. **运行项目**
   - 使用 HBuilderX 打开项目
   - 选择运行平台（H5、小程序、App等）
   - 点击运行按钮

### 配置说明

#### API服务器配置
API服务器默认运行在 `http://localhost:5000`，如需修改请编辑相关配置文件。

#### 数据库配置
项目支持两种数据存储方式：
- **SQLite**：用于App端本地存储
- **JSON**：用于轻量级数据存储

## 📁 项目结构

```
RailGo-UniApp/
├── api/                    # API服务器
│   ├── app.py             # Flask应用主文件
│   ├── requirements.txt   # Python依赖
│   ├── start.bat         # Windows启动脚本
│   └── start.sh          # Linux/Mac启动脚本
├── components/            # 自定义组件
├── pages/                # 页面文件
│   ├── index/            # 首页
│   ├── train/            # 车次查询
│   ├── station/          # 车站查询
│   ├── emu/              # 动车组查询
│   └── about/            # 关于页面
├── scripts/              # 工具脚本
│   ├── config.js         # 配置文件
│   ├── jsonDB.js         # JSON数据库
│   ├── sqlite.js         # SQLite数据库
│   └── req.js            # 网络请求
├── static/               # 静态资源
│   ├── icons/            # 应用图标
│   ├── trainHead/        # 车型图片
│   └── fonts/            # 字体文件
├── uni_modules/          # UniApp模块
├── manifest.json         # 应用配置
├── pages.json           # 页面配置
└── package.json         # 项目依赖
```

## 🔧 开发指南

### 添加新功能

1. **创建页面**：在 `pages/` 目录下创建新的页面文件
2. **注册路由**：在 `pages.json` 中注册新页面
3. **添加组件**：在 `components/` 目录下创建可复用组件
4. **数据处理**：在 `scripts/` 目录下添加相关工具函数

### 代码规范

- 使用 Vue 3 Composition API
- 遵循 ESLint 代码规范
- 组件命名使用 PascalCase
- 文件命名使用 kebab-case
- 添加必要的函数级别注释

### 调试技巧

- 使用 HBuilderX 内置调试工具
- 在真机上测试App功能
- 使用浏览器开发者工具调试H5版本
- 查看控制台日志排查问题

## 📊 数据来源

本项目的数据来源包括：

- **RailGo.Parser** - 车次、车站基础数据
- **MoeFactory车厢** - 正晚点、配属信息  
- **rail.re** - 动车组担当信息
- **车站小屏** - 车站大屏数据

所有数据均为计算机采集获得，仅供学习参考，不保证实时性、准确性或完整性。

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request 来帮助改进项目！

### 贡献流程

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建 Pull Request

### 贡献类型

- 🐛 Bug修复
- ✨ 新功能开发
- 📝 文档改进
- 🎨 UI/UX优化
- ⚡ 性能优化

## 📄 开源协议

本项目采用 [MIT License](LICENSE) 开源协议。

## 🙏 致谢

感谢以下项目和服务：

- [UniApp](https://uniapp.dcloud.io/) - 跨平台开发框架
- [Vue.js](https://vuejs.org/) - 渐进式JavaScript框架
- [FirstUI](https://www.firstui.cn/) - UniApp UI组件库
- [Flask](https://flask.palletsprojects.com/) - Python Web框架

## 📞 联系方式

- **项目主页**：[GitHub Repository](https://github.com/your-username/RailGo-UniApp)
- **问题反馈**：[Issues](https://github.com/your-username/RailGo-UniApp/issues)
- **功能建议**：[Discussions](https://github.com/your-username/RailGo-UniApp/discussions)

---

<div align="center">
  <p>Made with ❤️ by RailGo Team</p>
  <p>© 2025 RailGo. All rights reserved.</p>
</div>
