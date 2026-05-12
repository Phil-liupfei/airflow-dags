     
Git Sync 配置已经完成！现在需要提供 Git 仓库信息并应用配置。

## 下一步操作

### 1. **准备 Git 仓库**

创建一个专门存储 Airflow DAGs 的 Git 仓库，然后将现有 DAG 文件推送到仓库：

```bash
# 创建本地 DAGs 目录（如果还没有）
mkdir -p /home/phil/ldp-dags

# 复制现有 DAGs
cp -r /home/phil/ldp/airflow/dags/* /home/phil/ldp-dags/

# 初始化 Git 仓库
cd /home/phil/ldp-dags
git init
git add .
git commit -m "Initial DAGs"

# 添加远程仓库（替换为你的仓库地址）
git remote add origin https://github.com/your-username/airflow-dags.git
git push -u origin main
```

### 2. **更新 Terraform 变量**

在 `terraform/variables.tf` 或创建 `terraform.tfvars` 文件来设置你的 Git 仓库信息：

```bash
cat > /home/phil/ldp/terraform/terraform.tfvars << EOF
dags_git_repo     = "https://github.com/your-username/airflow-dags.git"
dags_git_branch   = "main"
dags_git_revision = "HEAD"
dags_git_subpath  = ""  # 如果 DAGs 在仓库根目录，留空
EOF
```

### 3. **应用配置**

```bash
cd /home/phil/ldp/terraform
terraform plan  # 检查配置
terraform apply -auto-approve  # 应用配置
```

### 4. **验证 Git Sync 状态**

```bash
# 查看 Git Sync pod 状态
kubectl get pods -n ldp -l app=airflow-git-sync

# 查看 Git Sync 日志
kubectl logs -n ldp -l app=airflow-git-sync --tail=50
```

### 5. **测试 DAG 同步**

```bash
# 修改本地 DAG 文件
echo "# Modified" >> /home/phil/ldp-dags/example_spark_job.py

# 提交并推送到仓库
git add .
git commit -m "Update DAG"
git push

# 等待约 60 秒（默认同步间隔）后检查
kubectl exec -n ldp -l component=scheduler -- airflow dags list
```

---

## 配置说明

| 配置项 | 说明 |
|--------|------|
| `dags_git_repo` | Git 仓库 URL |
| `dags_git_branch` | 分支名称（默认 main） |
| `dags_git_revision` | 版本（HEAD 或 commit hash） |
| `dags_git_subpath` | DAGs 在仓库中的子目录（可选） |
| `syncInterval` | 同步间隔（秒），默认 60 秒 |

这样配置后，每次你推送到 Git 仓库的 DAG 变更会自动同步到 Airflow，无需手动复制文件！