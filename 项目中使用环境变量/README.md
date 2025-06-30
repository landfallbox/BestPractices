# 项目中使用环境变量

如果项目中使用到一些需要用户或开发者自行设置的变量，如 API Key、token、Java安装路径等，推荐在项目中创建 `.env` 文件，将上述信息以 K-V 形式存在其中，使用 [python-dotenv](https://github.com/theskumar/python-dotenv) 在代码中读取定义的信息。

## 为什么使用 .env 存储环境变量

### 安全

`.env` 不会提交到 Git 上，减少敏感信息泄露的风险。这样的作法遵循了不在源代码中硬编码敏感信息的最佳实践。

### 便于管理

配置信息都集中在一个地方，方便管理和更新。

### 灵活

项目在不同环境（开发、测试、生产等）运行时，可以通过更换不同的 `.env` 文件来快速切换环境配置，无需修改代码本身。在多人协作和应用部署时尤为重要。

## Getting Started

安装 python-dotenv 包

```
pip install python-dotenv
```

## 使用

python-dotenv 从项目根路径下的 `.env` 文件中加载 K-V 变量。

`.env` 文件中内容定义为：

```
# Development settings
DOMAIN=example.org
ADMIN_EMAIL=admin@${DOMAIN}
ROOT_URL=${DOMAIN}/app
```

Python 代码中读取定义的变量值：

```python
from dotenv import load_dotenv

load_dotenv()  # take environment variables

# Code of your application, which uses environment variables (e.g. from `os.environ` or
# `os.getenv`) as if they came from the actual environment.
```

