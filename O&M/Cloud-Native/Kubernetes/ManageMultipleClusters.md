### **合并多个 kubeconfig 文件**

你可以将多个 `admin.conf` 文件合并到一个 kubeconfig 文件中，并使用 `kubectl config use-context` 切换不同的集群。

#### **步骤**

1. **设置 `KUBECONFIG` 变量**

   ```bash
   export KUBECONFIG=/path/to/admin1.conf:/path/to/admin2.conf:/path/to/admin3.conf
   ```

   这样 `kubectl` 就会同时加载多个配置文件。

2. **合并配置** 你可以手动合并多个 `admin.conf`，或者让 `kubectl` 帮你合并：

   ```bash
   kubectl config view --merge --flatten > ~/.kube/config
   ```

   这样所有的集群信息会被合并到 `~/.kube/config` 中。

3. **查看可用的 context**

   ```bash
   kubectl config get-contexts
   ```

4. **切换到指定集群**

   ```bash
   kubectl config use-context <context-name>
   ```
   


------

### **使用 `--kubeconfig` 选项**

如果你不想合并配置文件，可以使用 `--kubeconfig` 参数指定不同的 `admin.conf`。

#### **示例**

```bash
kubectl --kubeconfig=/path/to/admin1.conf get nodes
kubectl --kubeconfig=/path/to/admin2.conf get pods -A
```

这样可以临时指定 `kubeconfig` 文件，而不影响全局环境。
