# ingress 常见转发配置

### 1. 访问 *.xxxxx.com 转发到 *.m.xxxxx.com
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
 annotations:
   nginx.ingress.kubernetes.io/server-snippet: |
     if ($host ~* "(.*).xxxxx.com") {
          return 301 http://m.xxxxx.com/$1$request_uri ;
     }
 name: m-web
 namespace: default
spec:
 rules:
 - host: '*.xxxxx.com'
   http:
     paths:
     - backend:
         serviceName: m-web-nginx
         servicePort: 80
       path: /
 tls:
 - hosts:
   - m.qa.hxtrip.com
   secretName: m.qa-https
```
### 2. 访问 rewrite.bar.com/something 重定向到 rewrite.bar.com/
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```
### 3. 访问 rewrite.bar.com/ 重定向到 - rewrite.bar.com/api/
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
annotations:
  nginx.ingress.kubernetes.io/app-root: /api
name: approot
namespace: default
spec:
rules:
- host: approot.bar.com
  http:
    paths:
    - backend:
        serviceName: http-svc
        servicePort: 80
      path: /
```
### 4. 访问 host.example.com/service/ ⇒ app1:8080/。如果app1中服务路由并不是定义在根目录(/)，假设我们的实际情况是： host.example.com/service/ ⇒ app1:8080/s1/
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite /service/(.*)  /s1/$1 break;
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /service/
        backend:
          serviceName: app1
          servicePort: 8080
```
