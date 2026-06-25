"# WSL 调用 Windows Maven 编译指南

## 背景
在 WSL 中开发 Java 项目时，Maven Daemon (mvnd) 以独立的 Java 进程运行，会忽略脚本中设置的 JAVA_HOME。
- `mvnd` daemon 默认使用系统 Java（如 Java 17），导致编译时报 `java.lang.Record` 与 DataX 的 Record 冲突
- 需要使用 **普通 mvn**（非 daemon）+ **Java 8** 来避免兼容性问题

## Maven 位置
| 用途 | 路径 |
|------|------|
| .wsl-mvn.cmd（原始脚本，mvnd） | F:\Users\yzh\IdeaProjects\ytd\processor\.wsl-mvn.cmd |
| mvn.cmd（推荐） | F:\Program Files\maven-mvnd\mvn\bin\mvn.cmd |

## 方法一：PS1 脚本（推荐，最可靠）
```bash
cat > /mnt/c/tmp/build_project.ps1 << 'PS_EOF'
$env:JAVA_HOME = \"F:\Users\yzh\.jdks\azul-1.8.0_452\"
$env:PATH = \"F:\Users\yzh\.jdks\azul-1.8.0_452\bin;\" + $env:PATH
cd \"F:\Users\yzh\IdeaProjects\<project>\.worktrees\<branch>\"
& \"F:\Program Files\maven-mvnd\mvn\bin\mvn.cmd\" [参数]
PS_EOF
/mnt/c/Windows/SysWOW64/windowspowershell/v1.0/powershell.exe -ExecutionPolicy Bypass -Command \"& 'C:/tmp/build_project.ps1'\"
```

## 方法二：批处理 .bat + PowerShell 调用
```bash
cat > /mnt/c/tmp/build_mvn.bat << 'BATCH_EOF'
@echo off
set JAVA_HOME=F:\Users\yzh\.jdks\azul-1.8.0_452
set PATH=F:\Users\yzh\.jdks\azul-1.8.0_452\bin;%PATH%
cd /d F:\Users\yzh\IdeaProjects\<project>\.worktrees\<branch>
call \"F:\Program Files\maven-mvnd\mvn\bin\mvn.cmd\" [参数]
BATCH_EOF
/mnt/c/Windows/SysWOW64/windowspowershell/v1.0/powershell.exe -Command \"& 'C:/tmp/build_mvn.bat'\"
```

## 方法三：原始 .wsl-mvn.cmd（mvnd，无 Java 冲突时可用）
```bash
/mnt/c/Windows/SysWOW64/windowspowershell/v1.0/powershell.exe -Command \"& 'F:\Users\yzh\IdeaProjects\ytd\processor\.wsl-mvn.cmd' [参数]\"
```
> ⚠️ mvnd daemon 使用自身 Java（Java 17），不受 JAVA_HOME 控制

## 关键注意事项
### 路径引号
含空格的路径必须加双引号：`\"F:\Program Files\...\"`

### PowerShell 调用 WSL 文件的格式
- ✅ `& 'C:/tmp/build_mvn.bat'`（正斜杠）
- ❌ `& 'C:	mp\build_mvn.bat'`（反斜杠被转义）
- ❌ `& '/mnt/c/tmp/...'`（Windows 不认 WSL 路径）

### -D 参数问题
- ✅ 写 .ps1/.bat 脚本再执行
- ❌ 直接在 -Command 中嵌入 mvn 参数（-D 被 PowerShell 截断）

## Git Worktree
始终使用 **Windows Git** 创建 worktree：
```bash
# ✅ Windows Git
\"/mnt/f/Program Files/Git/cmd/git.exe\" worktree add .worktrees/<name> <branch>
# ❌ WSL git（Windows 端打不开）
git worktree add ...
```

## 常见问题排查
| 问题 | 原因 | 解决 |
|------|------|------|
| java.lang.Record 冲突 | mvnd 使用 Java 16+ | 改用 mvn（方法一/二） |
| jdk.tools:1.7 找不到 | 传递依赖指向不存在的 JDK | pom.xml 排除 jdk.tools:jdk.tools |
| No plugin found for prefix '.repo.local=F' | -D 被 PowerShell 截断 | 使用 .ps1 脚本 |
| Worktree Windows 端打不开 | 用 WSL git 创建 | 改用 Windows Git |
"