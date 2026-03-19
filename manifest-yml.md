### github-actions

```yaml
---
applications:
  - name: github-actions
    memory: 2048M    
    disk_quota: 4096M       # 硬盘给 1G (注意！ 硬盘大小要大于镜像大小)        
    random-route: true                
    docker:
      image: ghcr.io/workerspages/github-actions:github-actions-mariadb
    env:
      TZ: "Asia/Shanghai"       
      ADMIN_USER: "admin"
      ADMIN_PASSWORD: "admin"
      JWT_SECRET: "your_secret_key_change_me"
```
---
### alist-rclone

```yaml
---
applications:
  - name: alist-rclone
    memory: 2048M    
    disk_quota: 4096M       # 硬盘给 1G (注意！ 硬盘大小要大于镜像大小)        
    random-route: true                
    docker:
      image: ghcr.io/workerspages/alist-rclone:latest
    env:
      TZ: "Asia/Shanghai" 
```
---


### n8n

```yaml
---
applications:
  - name: n8n
    memory: 2048M    
    disk_quota: 4096M
    random-route: true                
    docker:
      image: blowsnow/n8n-chinese:latest
    env:
      TZ: "Asia/Shanghai" 
      N8N_SECURE_COOKIE: "false"
```
---
