# Flutter手册

量潮工作手册语言、框架和工具目录下的 Flutter 子手册，记录 Flutter 项目结构、组件命名、数据模型、私有包、CI、Web 部署、低码工具和多端测试。Flutter 手册的重点不是“怎么写 widget”，而是目标平台、工程结构、资源、状态、插件、测试和发布证据要一起闭环。

## 跨平台场景

- 移动端、Web、桌面端或多端复用的 Flutter 应用。
- 单一代码库覆盖多个平台，但需要区分平台能力和发布条件的项目。
- 需要统一管理 UI、状态、路由、资源、主题和构建流程的应用。
- 需要接入 Dart package、原生 plugin、签名证书和应用商店发布的场景。
- 需要将 Flutter Web 部署到 GitHub Pages 或云开发静态网站托管的场景。

## 应用工程原则

- 先确定目标平台和最低支持版本，再定工程结构。
- `pubspec.yaml`是工程入口，依赖、资源、字体和元数据都要保持干净。
- 先确定 widget、状态、路由和主题边界，再做页面细节。
- 平台差异要显式处理，不能靠“某个平台刚好能跑”。
- 测试、构建、签名和发布证据要在发布前闭环。
- 插件和原生能力接入要提前确认权限、兼容性和回退方案。

## 项目创建

Flutter 项目可以根据 Android Studio 创建应用的引导生成，也可以在命令行创建：

```bash
flutter create <your-dir>
```

项目结构至少说明：

- `pubspec.yaml`：依赖、资源、字体和元数据。
- `lib/`：入口和业务代码。
- `assets/`：图片、字体和静态资源。
- `test/`：单元测试和组件测试。
- `integration_test/`：集成测试。
- `android/`、`ios/`、`web/`、`macos/`、`windows/`、`linux/`：按目标平台使用。

## 配置和入口

源素材里将`pubspec.yaml`作为配置核心。所有依赖、资源、字体、插件和包信息都应通过它维护。

工程入口要明确：

- 主入口文件。
- 路由配置。
- 主题配置。
- 环境配置。
- 资源路径。

如果工程中有`main.dart`、`router.dart`、`theme.dart`、`config.dart`等文件，要说明各自责任，避免所有配置混在入口文件里。

## 组件命名

组件和模型命名要避免数据模型、视图组件、控制器组件、视图模型之间互相冲突。

命名规则：

- 数据模型：不加后缀，或使用`Model`后缀。
- 视图组件：使用`View`后缀。
- 控制器组件：使用`Controller`后缀。
- Provider 实现的视图模型组件：使用`Provider`后缀。

命名要使用具体名词描述单个实体，例如`User`、`Product`，避免无意义的复数和泛称。

## 数据模型

网络 API 返回值不要长期直接用`Map`和`List`在业务代码里传递。

推荐做法：

- 在`lib/models`下定义数据模型。
- 使用 Model 类约定字段和类型。
- 从请求数据创建 Model 实例。
- 业务代码访问 Model 属性，减少拼写错误和动态类型风险。

Model 抽象类至少需要：

- `fromMap`构造器：把`Map`转成 Model。
- `toMap`方法：把 Model 转回`Map`。
- 必要的字段校验或默认值处理。

跨组件使用 Model 时，可以结合 Provider 改造状态管理。

## 导航组件

侧边导航栏适用于导航单元较多的场景，例如：

- 管理后台功能导航。
- 文档目录。
- 超过 5 个导航项，普通移动端底部导航或桌面顶部导航不够用的场景。

必要时，侧边导航栏也可以作为主导航栏的二级导航。

## 私有包和插件

Flutter 私有 package 的源素材里给出几种方案。

### 公开 Git 地址

初步方案：

1. 在 Coding 设置项目访问为公开，并获取公开访问地址。
2. 在 Flutter 项目的`pubspec.yaml`里设置 Git 下载。
3. 参考 Dart 官方 Git packages 文档。

优点是简单，缺点是只能用于可以公开的包。

### SSH

SSH 可行，Dart 官方支持 Git package 通过 SSH 访问，Coding 也支持 SSH 密钥。

主要问题是开发者本地配置门槛较高，需要配置 SSH key 和访问权限。

### Deploy Token

Deploy Token 思路优雅，但源素材判断为不可行。

原因：

- GitLab/Coding 个人令牌可以访问代码仓库。
- 但 Flutter / Dart 的`pubspec.yaml`不支持注入环境变量。
- Dart 官方也不打算支持这种能力。

因此不能把私有 token 安全地写入`pubspec.yaml`。

### 本地 path package

本地打包可行且相对安全，但流程破碎。

适用场景：

- 本地联调。
- 临时开发。
- 尚未进入稳定分发的包。

不适合当作长期团队分发方式。

### 制品库

源素材判断 Coding 暂不支持相关制品库能力，因此当前不作为主方案。

## 低码和原型

FlutterFlow 可以作为低码或原型工具关注。

适用想法：

- 用低码平台替代 Sketch 等原型软件。
- 交给产品先出原型。
- 导出代码后由开发继续加工。

限制：

- FlutterFlow 的组件还不完整。
- 源素材中特别提到缺少`ExpansionPanelList`等需要的组件。
- 导出代码后仍要由开发检查结构、状态和可维护性。

## 测试

测试阶段确认应用不只是能打开。移动端、Web 和桌面端的布局、输入、权限和导航都要分别看。

本地测试：

```bash
flutter test
```

测试内容：

- widget test。
- unit test。
- integration test。
- 不同设备、分辨率和平台表现。
- 插件权限和原生能力。

设备兼容性测试可以考虑使用云计算提供的移动测试等工具，在云端调用多类型设备进行测试。

## CI 和 Web 部署

Flutter CI 可以通过 GitHub Actions 组织。

源素材里提到：

- `flutter-action`：提供 Flutter 环境。
- `flutter-gh-pages`：将 Flutter for Web 部署到 GitHub Pages。

Flutter Web 也可以部署到云开发静态网站托管。源素材理由是：方便做 DevOps 流程、有技术支持、团队更熟悉云开发。

CI 检查项：

- Flutter SDK 版本。
- 依赖安装。
- `flutter test`。
- Web 或目标平台构建。
- 构建产物上传或部署。
- 部署后访问验证。

## 包发布

发布 Dart / Flutter package 到 pub.dev 时，要注意国内代理镜像的影响。

如果本地通常把 pub.dev 镜像设置为国内代理，上传时需要增加`server`参数，指定上传服务器为官方源。

发布前确认：

- 包名、版本和描述。
- `pubspec.yaml`元数据。
- CHANGELOG。
- README。
- 示例代码。
- 发布目标是否为官方源。

## 应用类型

- 移动端业务应用：重点看权限、离线状态、安装包和应用商店要求。
- Web 控制台：重点看路由、响应式布局、加载速度和浏览器兼容。
- 桌面工具：重点看窗口尺寸、文件权限、系统菜单和安装方式。
- 跨平台原型：重点看复用边界和后续是否会变成正式产品。
- 内部演示应用：重点看展示路径、数据隔离和临时功能清理。

## 应用交付标准

- 支持平台、最低版本和发布渠道清楚。
- 启动入口、路由结构和主要页面清楚。
- `pubspec.yaml`、资源、字体、依赖和插件说明清楚。
- Model、View、Controller、Provider 命名边界清楚。
- 私有 package 方案明确，不把 token 写进`pubspec.yaml`。
- 单元、组件或集成测试覆盖关键流程。
- GitHub Actions、GitHub Pages 或云开发部署流程可复现。
- 兼容性风险、回退方式和发布证据清楚。

## 发布前检查

- `flutter doctor`是否通过
- 是否使用`flutter create`或等价流程初始化
- `pubspec.yaml`是否整理好
- 入口文件、路由、主题和配置是否清楚
- Model 是否替代裸`Map`长期流转
- 组件命名是否区分 Model/View/Controller/Provider
- 私有包方案是否明确
- 是否有 widget 和集成测试
- 是否明确 build target、签名和发布渠道
- 是否记录插件兼容性
