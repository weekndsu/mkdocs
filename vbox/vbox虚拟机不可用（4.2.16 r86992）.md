# vbox虚拟机不可用（4.2.16 r86992）

### 显示异常Message如下：

```
F:\tinderbox\win-4.2\src\VBox\Main\src-server\MachineImpl.cpp[725] (long __thiscall Machine::registeredInit(void)).　
```

### 解决步骤

- 切换至C:\Users\xxxx\VirtualBox VMs\ansible_jenkins

  ```
  ansible_jenkins.vbox-tmp
  ansible_jenkinsv.box-prev
  ```

- 把两个文件做个备份

- ansible_jenkins.vbox-tmp文件名更改为ansible_jenkins.vbox

- 重启vbox后恢复。
