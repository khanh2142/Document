# MIGRATE CRA TO VITE

Để tránh xảy ra conflig, chúng ta sẽ dựng từ 1 folder trống

```properties
npm create vite@latest
```

Sau khi câu lệnh được thực hiện, thông báo sẽ hiện lên và chúng ta sẽ điền các thông tin như sau:

```properties
Project name: // Tên của project
Select a framework: // Chọn framework (Ở đây chúng ta chọn React + Typescript)
```

Như vậy là đã cài xong `Vite` thành công
Tiếp theo, chúng ta sẽ đi đến folder `Vite` vừa setup

```properties
cd // Tên project vừa đặt ở trên
npm i // Cài đặt các package còn thiếu
```

Tiếp theo, chúng ta đến với phần mirgate từ `CRA` sang `Vite`

Đầu tiên, ta sẽ xóa thư mục `src` của project `Vite` đi.
Sau đó, copy thư mục `src` của project `CRA` sang.

Sau khi copy xong, chúng ta cần config lại tất cả đường dẫn thư mục.

Đối với file `tsconfig.json`, chúng ta config như sau:

```TSX
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "allowJs": false, // lưu ý rằng khi false tất cả tập tin đuôi .js hay .jsx đều báo lỗi (chỉ sử dụng .ts và .tsx)
    "skipLibCheck": true,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": "src", // config url gốc là thư mục src
    "paths": {
      "*": ["*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

Phần quan trọng tiếp theo là config lại file `vite.config.ts`:

```TSX
import react from "@vitejs/plugin-react";
import path from "path";
import { defineConfig } from "vite";
import viteRewriteAll from "vite-plugin-rewrite-all"; // npm i vite-plugin-rewrite-all
import svgrPlugin from "vite-plugin-svgr"; // npm i vite-plugin-svgr
import viteTsconfigPaths from "vite-tsconfig-paths"; // npm i vite-tsconfig-paths

// Nếu chưa có 3 plugin ở trên thì chúng ta sẽ chạy 3 câu lệnh cài ở trên

export default defineConfig(({ command, mode, ssrBuild }) => {
  return {
    // khi build xong, bản build sẽ không log hoặc debug
    esbuild: {
      drop: command === "build" ? ["console", "debugger"] : [],
      pure:
        command === "build"
          ? [
              "console.log",
              "console.error",
              "console.warn",
              "console.debug",
              "console.trace",
            ]
          : [],
    },
    // config các plugin cần thiết để khởi chạy
    plugins: [react(), viteRewriteAll(), viteTsconfigPaths(), svgrPlugin()],
    server: {
      port: 8001, // config port trên local
    },
    resolve: { // config lại đường dẫn
    // đối với các folder ngoài cùng của thư mục src, chúng ta sẽ config ở alias
      alias: {
        store: path.resolve(__dirname, "./src/store"),
        components: path.resolve(__dirname, "./src/components"),
        hooks: path.resolve(__dirname, "./src/hooks"),
        utils: path.resolve(__dirname, "./src/utils"),
        services: path.resolve(__dirname, "./src/services"),
        routes: path.resolve(__dirname, "./src/routes"),
      },
    },
  };
});
```

Khi config xong, chúng ta sẽ phải chỉnh sửa lại các file chứa biến môi trường `.env`.

Thay vì `REACT_APP` thì ta sẽ chuyển thành `VITE`:

```properties
REACT_DOMAIN=http://localhost:8001 // cra
->
VITE_DOMAIN=http://localhost:8001 // vite
```

Và nếu trong folder `src` vừa copy sang có những file sử dụng các biến môi trường `.env`, ta sẽ sửa lại như sau:

```properties
process.env.REACT_APP_ACC_BASE_URL // cra
->
import.meta.env.VITE_ACC_BASE_URL // vite
```

Một bước nữa, chúng ta cần cần phải config lại đường dẫn của file `index.tsx` tới file `index.html`.

Mặc định của `Vite` là:

```html
<script type="module" src="/src/main.tsx"></script>
```

Nếu chúng ta dùng `index.tsx` thì sẽ sửa lại như sau:

```html
<script type="module" src="/src/index.tsx"></script>
```

Cuối cùng, thêm hoặc sửa nội dung của file `vite-env.d.ts` như sau:

```properties
/// <reference types="vite/client" />
// cấu hình Vite và các plugin của nó hoạt động bình thường
/// <reference types="vite-plugin-svgr/client" />
// cho phép sử dụng chức năng của gói svgr, được sử dụng để tối ưu hóa và chuyển đổi tệp SVG thành thành phần React
```

Và khi mirgate sang chúng ta cần để ý các `package` được cài trước đó. Copy các `package cũ` sang `package mới`. Nếu `package mới` mà đã tồn tại 1 thư viện (vd : `react` , `react-dom`) thì ta sẽ cân nhắc `giữ lại` hoặc để về `bản cũ`)

Thường ta sẽ `giữ nguyên` khi cài xong `Vite`, tránh `config`.

Khi copy xong ta chạy câu lệnh:

```properties
npm i
```

Trong `package.json`, ta thêm câu lệnh `start` vào trong `script`:

```json
"script" : {
    "start" : "vite"
}
```

Cuối cùng, chạy câu lệnh:

```properties
npm start
```

_Tham khảo thêm tại: https://vitejs.dev/config/_
