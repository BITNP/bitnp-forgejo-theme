# BITNP Forgejo 主题

[git.bitnp.net](https://git.bitnp.net) 使用的 Forgejo 主题，适用于 Forgejo 17。

## 目录结构

```text
templates/                  覆盖 Forgejo 自带的页面模板
  home.tmpl                 首页
  repo/ org/ user/ shared/  仓库、组织、个人页面
public/assets/
  css/theme-bitnp-light.css 浅色配色
  css/theme-bitnp-dark.css  深色配色
  css/theme-bitnp-auto.css  跟随系统切换浅色或深色
  css/bitnp.css             两套配色共用的布局和组件样式
  img/                      Logo 和网站图标
  fonts/                    Geist 字体（OFL 许可，见 fonts/OFL.txt）
```

## 部署

1. 把 `templates/` 和 `public/` 复制到 Forgejo 的自定义目录。官方 rootless 镜像的路径是 `/var/lib/gitea/custom`，普通镜像是 `/data/gitea`。
2. 在 `app.ini` 中设置：

   ```ini
   [ui]
   DEFAULT_THEME = bitnp-auto
   THEMES = bitnp-auto,bitnp-light,bitnp-dark,forgejo-auto
   ```

3. 重启 Forgejo。修改模板后必须重启才会生效；只改 CSS 时刷新浏览器缓存即可。

## 本地预览

在仓库根目录运行：

```sh
mkdir -p .preview/custom/conf
podman run --rm --name bitnp-preview \
  --userns keep-id --user "$(id -u):$(id -g)" \
  -p 127.0.0.1:3000:3000 \
  -v "$PWD/.preview:/var/lib/gitea" \
  -v "$PWD/templates:/var/lib/gitea/custom/templates:ro" \
  -v "$PWD/public:/var/lib/gitea/custom/public:ro" \
  -e 'FORGEJO____APP_NAME=BITNP: 网络无限 开拓不懈' \
  -e FORGEJO__ui__DEFAULT_THEME=bitnp-auto \
  -e FORGEJO__ui__THEMES=bitnp-auto,bitnp-light,bitnp-dark,forgejo-auto \
  -e FORGEJO__security__INSTALL_LOCK=true \
  -e FORGEJO__database__DB_TYPE=sqlite3 \
  codeberg.org/forgejo/forgejo:17-rootless
```

然后打开 <http://127.0.0.1:3000>。需要查看登录后的页面时，另开终端创建测试账号：

```sh
podman exec bitnp-preview forgejo admin user create \
  --username demo --password 'Preview-pass-123' \
  --email demo@example.invalid --admin --must-change-password=false
```

预览数据保存在 `.preview/`，不会提交到仓库，可以随时删除。用 Docker 时把 `podman` 换成 `docker`，并去掉 `--userns keep-id`。

## 修改说明

- **调整配色**：改 `theme-bitnp-light.css` 和 `theme-bitnp-dark.css` 中的颜色变量。浅色主色 `#b4570e` 比官网 Logo 橙 `#da751d` 深，这样按钮上的白字才看得清。改主色时，按钮文字和背景的对比度应不低于 4.5:1。
- **调整布局**：改 `bitnp.css`，样式类名统一以 `bitnp-` 开头。
- **修改首页**：改 `templates/home.tmpl`，首页标题取自 `app.ini` 的 `APP_NAME`。
- **升级 Forgejo**：`templates/` 下的文件会整体替换 Forgejo 自带的同名模板。升级大版本前，先对照新版本的同名模板，把上游改动合并进来，再用本地预览检查一遍。
