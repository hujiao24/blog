---
categories:
- 建站技能
date: '2025-10-09 18:54:24+08:00'
description: ''
draft: false
image: ''
slug: migrate-domain
cover: /archives/migrate-domain/1qhbs4.png
tags:
- wordpress
title: 基于wordpress站点的域名迁移
---

需要对部署的 wordpress 站点进行域名迁移，从旧域名 `www.xxx.com` 迁移到新域名 `www.yyy.com`

在迁移前，旧的域名访问正常

1、在域名服务商站点中修改域名 `www.yyy.com`域名指向服务器地址，不同的域名服务商修改方式相差不大，增加 @ 主机记录和 www 主机记录

修改后域名生效需要一点时间，可以通过 ping 域名来确认 DNS 配置是否已经生效

![](/archives/migrate-domain/2xmzu9.png)


2、证书申请，因为证书是基于域名颁发的，在域名更换的时候，需要对新域名进行证书申请

之前有写过签发证书的，可以参考这里 https://blog.csdn.net/weixin_53109623/article/details/146275962 

分别运行下面两个命令

```shell
certbot --nginx -d yyy.com -d www.yyy.com --register-unsafely-without-email 

certbot renew --dry-run
```

3、修改 nginx 的配置信息，将 `xxx.com` 域名修改为 `yyy.com`域名，可以使用 `sed -i 's/xxx/yyy/g' nginx.conf` 进行替换

修改后的配置信息参考如下

```shell
server {
    listen 443 ssl;
    server_name yyy.com www.yyy.com;

    ssl_certificate /etc/letsencrypt/live/yyy.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yyy.com/privkey.pem; # managed by Certbot

    root /root/nginx/html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
        index index.php;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

在 nginx 对新的配置加载生效后，这时候访问旧的域名 `www.xxx.com` 已经访问不到了，访问新的域名 `www.yyy.com` 是可以访问

但是 wordpress 数据库中的配置还没修改，因此看到的是错乱的页面内容

4、修改 wordpress 的数据库，终端登录 mysql 连数据库，查询 wp_options 表达记录并修改

```sql
select * from wp_options where option_name = 'siteurl';
select * from wp_options where option_name = 'home';
select * from wp_options where option_name = 'admin_email';

update wp_options set option_value = 'https://www.yyy.com' where option_name = 'siteurl';
update wp_options set option_value = 'https://www.yyy.com' where option_name = 'home';
update wp_options set option_value = 'admin@yyy.com' where option_name = 'admin_email';

select * from wp_users where user_login = 'admin';

update wp_users set user_url = replace(user_url, 'xxx', 'yyy');
update wp_users set user_email = replace(user_email, 'xxx', 'yyy');

select * from wp_usermeta where meta_key = 'user_custom_avatar';
update wp_usermeta set meta_value = replace(meta_value, 'xxx', 'yyy') where meta_key = 'user_custom_avatar';

delete from wp_options where option_name = 'new_admin_email';
delete from wp_options where option_name = 'adminhash';
```

文章表 wp_posts 中的 post_content 和 guid 字段也需要一并修改域名为 `yyy.com`

```sql
update wp_posts set post_content = replace(post_content, 'xxx', 'yyy');
update wp_posts set guid = replace(guid, 'xxx', 'yyy');
```

5、修改完数据库后，这时候新的域名访问已经可以正常了，访问 `https://www.yyy.com/wp-login.php` 登录后台

登录的用户名密码还是和 `https://www.xxx.com/wp-login.php` 一样的，里面有部分配置值是需要修改的，比如一些菜单的链接地址

![](/archives/migrate-domain/1qhbs4.png)


6、如果旧域名设计了 logo 的话，也需要同步修改，另外对旧域名访问的时候显示一个页面迁移提示，并配置域名跳转，参考代码如下

```html
<!doctype html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>域名迁移提示</title>
  <script>
    const NEW_DOMAIN = "https://www.yyy.com";
    const REDIRECT_SECONDS = 3;
  </script>
  <style>
    :root{
      --bg1: #0f172a;
      --bg2: #0ea5e9;
      --card: rgba(255,255,255,0.06);
      --glass: rgba(255,255,255,0.06);
      --accent: #38bdf8;
      --text: #e6f7ff;
    }
    html,body{height:100%;margin:0;font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;}
    body{
      background: radial-gradient(1200px 600px at 10% 10%, rgba(14,165,233,0.12), transparent),
                  radial-gradient(1000px 500px at 90% 90%, rgba(14,165,233,0.08), transparent),
                  linear-gradient(180deg,var(--bg1), #062234 80%);
      color:var(--text);
      display:flex;align-items:center;justify-content:center;padding:24px;
    }
    .card{
      width:100%;max-width:720px;background:linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
      border-radius:16px;padding:28px;box-shadow: 0 10px 30px rgba(2,6,23,0.6);backdrop-filter: blur(6px);
      border:1px solid rgba(255,255,255,0.04);
      display:flex;gap:20px;align-items:center;
    }
    .logo{
      min-width:84px;height:84px;border-radius:12px;display:flex;align-items:center;justify-content:center;
      background:linear-gradient(135deg,var(--accent), #0ea5e9);font-weight:700;font-size:20px;color:#04293a;
      box-shadow: 0 6px 18px rgba(14,165,233,0.12);
    }
    .content{flex:1}
    h1{margin:0 0 8px 0;font-size:20px;letter-spacing:0.2px}
    p{margin:0 0 12px 0;color:rgba(230,247,255,0.88)}
    .meta{font-size:14px;color:rgba(230,247,255,0.72)}
    .actions{margin-top:16px;display:flex;gap:12px;align-items:center}
    .btn{
      padding:10px 14px;border-radius:10px;border:0;font-weight:600;cursor:pointer;font-size:14px;
      background:linear-gradient(90deg,#22c1c3,#0ea5e9);color:#00232a;box-shadow:0 8px 20px rgba(14,165,233,0.12);
    }
    .link{background:transparent;border:0;color:var(--text);text-decoration:underline;cursor:pointer;font-size:14px}
    .countdown{font-weight:700}
    .small{font-size:13px;color:rgba(230,247,255,0.6)}

    @media (max-width:560px){
      .card{flex-direction:column;align-items:flex-start;padding:20px}
      .logo{width:64px;height:64px;font-size:18px}
    }
  </style>
</head>
<body>
  <div class="card" role="region" aria-label="域名迁移提示">
    <div class="logo" aria-hidden="true">迁</div>
    <div class="content">
      <h1>站点已迁移到新域名</h1>
      <p class="meta">您正在被自动重定向到新域名。为保证访问体验，页面将在 <span id="seconds">3</span> 秒后跳转。</p>
      <div class="actions">
        <button id="goNow" class="btn">立即前往</button>
        <button id="cancel" class="link">取消跳转</button>
        <div style="margin-left:auto;text-align:right">
          <div class="small">目标： <a id="targetLink" href="#" style="color:inherit;text-decoration:underline"></a></div>
          <div class="small">如果浏览器没有自动跳转，请点击“立即前往”。</div>
        </div>
      </div>
    </div>
  </div>

  <script>
    (function(){
      const secondsEl = document.getElementById('seconds');
      const goNowBtn = document.getElementById('goNow');
      const cancelBtn = document.getElementById('cancel');
      const targetLink = document.getElementById('targetLink');

      targetLink.textContent = NEW_DOMAIN.replace(/^https?:\/\//, '');
      targetLink.href = NEW_DOMAIN;
      targetLink.target = '_blank';

      let remaining = REDIRECT_SECONDS;
      secondsEl.textContent = remaining;

      const interval = setInterval(()=>{
        remaining -= 1;
        if(remaining <= 0){
          secondsEl.textContent = 0;
          clearInterval(interval);
        } else {
          secondsEl.textContent = remaining;
        }
      },1000);

      let redirected = false;
      const redirectTimer = setTimeout(()=>{
        redirected = true;
        window.location.href = NEW_DOMAIN;
      }, REDIRECT_SECONDS * 1000);

      goNowBtn.addEventListener('click', function(){
        if(!redirected){
          clearTimeout(redirectTimer);
          window.location.href = NEW_DOMAIN;
        }
      });

      cancelBtn.addEventListener('click', function(){
        clearTimeout(redirectTimer);
        clearInterval(interval);
        secondsEl.textContent = '已取消';
        cancelBtn.textContent = '跳转已取消';
        cancelBtn.disabled = true;
      });

      try{
        const params = new URLSearchParams(location.search);
        if(params.has('to')){
          const t = params.get('to');
          if(/^https?:\/\//.test(t)){
            targetLink.href = t;
            targetLink.textContent = t.replace(/^https?:\/\//,'');
            clearTimeout(redirectTimer);
            setTimeout(()=>{ window.location.href = t; }, REDIRECT_SECONDS * 1000);
          }
        }
      }catch(e){/* ignore */}
    })();
  </script>
</body>
</html>
```

