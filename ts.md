## TypeScript 项目初始化步骤

在本地从零开始初始化一个全新的 TypeScript 项目，可以遵循以下核心步骤：

1. **新建并进入项目目录**：
   ```bash
   mkdir my-ts-app
   cd my-ts-app
   ```
2. **初始化 Node/NPM 环境**：
   ```bash
   npm init -y
   ```
3. **安装 TypeScript 核心依赖（作为开发依赖）**：
   ```bash
   npm i -D typescript @types/node
   ```
4. **自动生成基础 `tsconfig.json` 配置文件**：
   ```bash
   npx tsc --init
   ```
5. **创建存放源码的目录**：
   ```bash
   mkdir src
   ```

## 编写与运行 TypeScript 基本示例

1. **编写入口源码文件**：
   创建 `src/index.ts` 文件，编写以下基本的 TypeScript 逻辑：
   ```ts
   const msg: string = "Hello TypeScript";
   console.log(msg);
   ```

2. **配置自动化脚本**：
   在项目根目录下的 `package.json` 中配置编译与运行的 `scripts` 脚本：
   ```json
   {
     "scripts": {
       "build": "tsc",
       "start": "node dist/index.js",
       "dev": "ts-node src/index.ts"
     }
   }
   ```

3. **运行与编译命令**：
   * **开发阶段热运行**（直接执行内存中的 TS 代码）：
     ```bash
     npm run dev
     ```
   * **生产构建编译**（将 TS 编译为 `dist` 目录下的 JS 代码）：
     ```bash
     npm run build
     ```
   * **生产运行**（运行编译后的原生 JS 逻辑）：
     ```bash
     npm run start
     ```
