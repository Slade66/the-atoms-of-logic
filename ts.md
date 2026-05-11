
### 初始化 TS 项目

```bash
# 1) 新建并进入项目目录
mkdir my-ts-app
cd my-ts-app

# 2) 初始化 npm
npm init -y

# 3) 安装 TypeScript（开发依赖）
npm i -D typescript @types/node

# 4) 生成 tsconfig.json
npx tsc --init

# 5) 创建源码目录和入口文件
mkdir src
```

编写一个 `src/index.ts`
```ts
const msg: string = "Hello TypeScript";
console.log(msg);
```

在 package.json 加脚本：
```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts"
  }
}
```

```bash
npm run build
npm run start
# 或开发时
npm run dev
```
