---
title: Webpack -> Vite Migration
author: yong
date: 2023-02-07 11:58:00 +0800
categories: [React, FE Tech]
tags: [React, Vite, Webpack, Bundler]
---

# AD Platform Webpack → Vite Migration

회사에서 기존 AD Platform에서 새로운 Feature를 개발하는 도중, 프로젝트 규모 자체가 커서 그런지 HMR과 서버를 재시작하거나 빌드할 때 시간이 상당히 오래 걸렸다. 그 전에 다른 프로젝트에서 Vite를 도입했던 기억이 떠올라 이번엔 Webpack에서 Vite로 마이그레이션을 해보기로 했다.

# ⚙️ 기존 환경

| Dependencies | Version |
| ------------ | ------- |
| react        | 17.0.2  |
| webpack      | 4.42.0  |

# 🔄 Migration

## 🔹 Vite Install

Vite에서 React 플러그인 기능을 사용하기 위해 다음과 같이 플러그인을 설치했다.

```bash
yarn add -D vite @vitejs/plugin-react
```

설치 후, 프로젝트 루트에 `vite.config.ts` 파일과 `index.html` 파일을 생성하여 다음과 같이 설정한다.

```tsx
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import envCompatible from "vite-plugin-env-compatible";

export default defineConfig({
  plugins: [react(), envCompatible({ prefix: "REACT_APP" })],
  publicDir: false,
});
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    ...
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./src/index.tsx"></script>
  </body>
</html>
```

`index.html` 파일을 public 디렉터리가 아닌 루트에 생성한 이유는 Vite의 의도적인 구조이다. 추가적인 번들링 과정 없이 `index.html` 파일이 앱의 진입점이 되기 위한 의도이다.

```bash
yarn add -D vite-plugin-env-compatible
```

Vite에선 환경 변수에 접근할 때, `process.env` 대신 `import.meta.env` 를 통해 접근한다. 하지만 기존 프로젝트에서의 환경 변수를 Vite의 환경 변수 접근 방식으로 수정하려면 리소스를 많이 소요하게 된다. 따라서 기존 `process.env` 을 통해 접근할 수 있도록 도와주는 플러그인을 설치한다.

## 🔹 Proxy

Vite에서 프록시는 `vite.config.ts` 에서 설정할 수 있다. [참고](https://vitejs-kr.github.io/config/server-options.html#server-proxy)

`vite.config.ts` 파일에서 환경 변수를 통해 프록시를 다음과 같이 설정하였다.

```tsx
...

export default ({ mode }) => {
	const envDir = `../environments`
  process.env = { ...process.env, ...loadEnv(mode, envDir, '') }

	...
	return defineConfig({
		...
		server: {
	    proxy: {
	      '/auth': {
	        target: `${AuthURL}`,
	        changeOrigin: true,
	        rewrite: path => path.replace(/^\/api/, ''),
	      },
				...
			}
		},
		preview: {
	    proxy: {
	      '/auth': {
	        target: `${AuthURL}`,
	        changeOrigin: true,
	        rewrite: path => path.replace(/^\/api/, ''),
	      },
				...
			}
		}
	})
}
```

# 🚀 Trouble Shooting

## 🔹 The following dependencies are imported but could not be resolved

기본적인 Vite 설정 완료 후, 실행 했더니 다음과 같이 에러가 나타났다.

<img src="/webpack-vite-1.png" alt="webpack-vite-1.png">

이는 Vite의 모듈 번들러인 Rollup의 파일 탐색 방식 때문인데, rollup 설정의 `build.rollupOptions.external` 이 외부 파일보다 `/` 을 먼저 찾기 때문에 발생한 오류였다.

```html
<body>
  <div id="root"></div>
  <script type="module" src="./src/index.tsx"></script>
</body>
```

기존의 `index.html` 파일에서는 위와 같이 설정되어 있었다. 아래처럼 `.` 를 제거하니 정상적으로 실행이 완료되었다.

```html
<body>
  <div id="root"></div>
  <script type="module" src="/src/index.tsx"></script>
</body>
```

## 🔹 Internal server error: Failed to resolve import "…" from "src/index.tsx". Does the file exist?

<img src="/webpack-vite-2.png" alt="webpack-vite-2.png">

서버가 정상적으로 실행되는 듯 했으나 이번에도 경로를 찾지 못해 파일을 불러오지 못하는 에러가 발생했다. 프로젝트 내부에서 모든 파일 코드를 수정하기엔 큰 리소스 소모가 필요해보였다. 이는 vite-tsconfig-paths 플러그인을 설치하여 해결하였다. 해당 플러그인은 TypeScript의 경로 매핑을 그대로 사용할 수 있게끔 도와주는 플러그인이다.

```html
yarn add -D vite-tsconfig-paths
```

그리고 `vite.config.js` 파일을 다음과 같이 수정하였다.

```tsx
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tsconfigPaths from "vite-tsconfig-paths";
import envCompatible from "vite-plugin-env-compatible";

export default defineConfig({
  plugins: [react(), tsconfigPaths(), envCompatible({ prefix: "REACT_APP" })],
  publicDir: false,
  base: "/",
});
```

## 🔹 Require is not defined

Vite는 기본적으로 ES Module 방식을 사용하기 때문에, require 문(CJS)을 사용하면 `require is not defined` 에러를 출력한다.

<img src="/webpack-vite-3.png" alt="webpack-vite-2.png">

<img src="/webpack-vite-4.png" alt="webpack-vite-4.png">

이는 vite 프로젝트에서 require를 사용할 수 있게끔 도와주는 플러그인을 설치하여 해결하였다. [링크](https://www.npmjs.com/package/vite-plugin-require)

```bash
yarn add vite-plugin-require
```

```tsx
...
import vitePluginRequire from 'vite-plugin-require'

export default defineConfig({
  plugins: [
		...
		vitePluginRequire({
      fileRegex: /.ts$|.tsx$|.jpg$|.png$/,
    }),
		...
	],
  ...
})
```

## 🔹 Please import the top-level fullcalendar lib before attempting to import a plugin.

해당 이슈는 fullcalendar 디펜던시 버전을 업데이트하며 해결되었다.

이 외에도 여러가지 자질구레한 플러그인 버전 충돌, 경로 오타(이거 때문에 하루 종일 삽질) 등등 이슈가 많이 나타났었다.

# 🌟 결과

## 🔹 Hot Module Reload

- Webpack
<br>
  <img src="/HMR-Webpack.gif" alt="HMR-Webpack">

- Vite
<br>
  <img src="/HMR-Vite.gif" alt="HMR-Vite">

## 🔹 Build Time

- Webpack: 45s
- Vite: 24s

## 🔹 Cold Start

- Webpack: 약 16s
- Vite: 약 4s

아직 전체적으로 테스트를 해보진 못했지만, 속도가 비약적으로 향상되어 앞으로 개발 생산성도 같이 향상 될 것으로 기대하고 있다.
