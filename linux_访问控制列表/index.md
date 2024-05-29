# Linux 上的访问控制列表


Linux 上的访问控制列表 (acl, access control list) 为文件系统提供了一种额外的、更灵活的权限机制。相比于使用 chmod 修改文件的模式比特位以进行访问控制, acl 可以指定具体的某个用户或者某个组对文件的访问权限。

## 安装和启用 acl
在 Archlinux 和 CentOS 中 acl 包是默认安装的, Debian 中需要: `sudo apt install acl`.

acl 包中的 `getfacl`:用于显示文件的 acl, `setfacl`:用于设置文件的 acl.

在大部分的文件系统中(ext2/3/4, xfs, btrfs...), acl 都是默认启用的。

如果当前系统系统未启用 acl, 除了使用对应的文件系统管理工具设置 acl 参数外, 还可以在 `/etc/fstab`:挂载选项中加上 acl 参数, 例如: `UUID=...   /   ext4    rw,relatime,acl	0 1`.

## acl 的使用
### getfacl 显示 acl
`getfacl [-aceEsRLPtpndvh] file ...`
- `-a, --access`: 仅显示文件的访问控制列表
- `-d, --default`: 仅显示默认的访问控制列表
- `-c, --omit-header`: 不现实头部注释
- `-e, --all-effective`: 打印所有有效全线
- `-E, --no-effective`: 打印无效权限
- `-s, --skip-base`: 跳过仅有基本项的文件
- `-R, --recursive`: 递归
- `-L, --logical`: 跟踪符号链接
- `-P, --physical`: 不跟踪符号链接
- `-t, --tabular`: 使用表格的形式打印
- `-n, --numeric`: 打印用户/组标识符
- `--one-file-system`: 跳过不同文件系统的文件
- `-p, --absolute-names`: 不去除路径名中前导的'/'
- `-v, --version`: 版本
- `-h, --help`: 帮助

一般只需要 `getfacl file` 显示 acl, 若要查看目录中所有文件的 acl 可以使用 `getfacl -Rs dir`.

acl 中的基本条目与 ugo/rwx 相对应, 不可删除, 形如:
```
user::rw-
group::r--
other::r--
```

### setfacl 设置 acl
`setfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...`
- `-m, --modify=acl`: 修改文件当前的 acl 条目
- `-M, --modify-file=file`: 从文件中读取要修改的 acl 条目
- `-x, --remove=acl`: 删除文件指定的 acl 条目
- `-X, --remove-file=file`: 从文件中读取要删除的 acl 条目
- `-b, --remove-all`: 删除所有扩展的 acl 条目
- `-k, --remove-default`: 删除默认的 acl
- `--set=acl`: 设置并替换文件的 acl, 替换当前的 acl
- `--set-file=file`: 从文件中读取要设置的 acl 条目
- `--mask`: 请重新计算有效的权限掩码
- `-n, --no-mask`: 不要重新计算有效的权限掩码
- `-d, --default`: 操作应用于默认 acl
- `-R, --recursive`: 递归
- `-L, --logical`: 跟踪符号链接
- `-P, --physical`: 不跟踪符号链接
- `--restore=file`: 恢复 acl (与 "getfacl-R" 相反)
- `--test`: 测试模式 (不修改 acl)
- `-v, --version`: 版本
- `-h, --help`: 帮助

为用户设置 acl: `setfacl -m "u:user:permissions" <file/dir>`

为组设置 acl: `setfacl -m "g:group:permissions" <file/dir>`

为其他人设置 acl: `setfacl -m "other:permissions" <file/dir>`

其中 user/group 可以是用户名/组名或 ID, permissions 是形如 `r-x` 的权限。

整个形如 `"u:user:permissions"` 称为一个条目 (entry).

为指定目录下的新创建的文件或目录设置默认的条目: `setfacl -dm "entry" <dir>`, 默认的条目不会影响移动到目录中的文件。

删除默认条目: `setfacl -k <file/dir>`.

删除所有扩展的条目: `setfacl -b <file/dir>`.
