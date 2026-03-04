# 🏗️ Civil Engineer Web — Dockerized & k3s Deployment

Sitio web profesional para estudio de ingeniería civil. Stack: **HTML/CSS/JS + nginx**, dockerizado y desplegado en **k3s** con **ArgoCD** (GitOps).

---

## 📁 Estructura del proyecto

```
civil-engineer-web/
├── frontend/
│   └── index.html              # Sitio estático completo
├── .github/
│   └── workflows/
│       └── docker-build.yml    # CI: build & push a GHCR
├── k8s/
│   ├── namespace.yaml          # Namespace: civil-web
│   ├── deployment.yaml         # 2 réplicas, rolling update
│   ├── service.yaml            # ClusterIP interno
│   └── ingress.yaml            # Traefik ingress + redirect HTTPS
├── argocd/
│   └── application.yaml        # ArgoCD Application (GitOps)
├── Dockerfile                  # nginx:alpine + sitio
├── nginx.conf                  # Config nginx con gzip y headers
└── README.md
```

---

## 🚀 Setup rápido

### 1. Configurar variables en los archivos

Reemplazá los placeholders en los manifiestos:

| Archivo | Placeholder | Valor |
|---|---|---|
| `k8s/deployment.yaml` | `YOUR_GITHUB_USER` | tu usuario de GitHub |
| `k8s/ingress.yaml` | `obras.tudominio.com.ar` | tu dominio real |
| `argocd/application.yaml` | `YOUR_GITHUB_USER/YOUR_REPO_NAME` | tu repo |

### 2. Push al repo

```bash
git init
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git add .
git commit -m "feat: initial civil engineer web"
git push -u origin main
```

El **GitHub Action** se dispara solo en cada push a `main` y:
- Buildea la imagen Docker
- La pushea a GHCR (`ghcr.io/tu-usuario/repo:latest`)
- Actualiza el tag en `k8s/deployment.yaml` y hace commit

### 3. Registrar la app en ArgoCD

```bash
# Aplicar el Application manifest en el cluster
kubectl apply -f argocd/application.yaml

# Ver estado en ArgoCD
kubectl -n argocd get app civil-engineer-web
```

ArgoCD detecta cambios en el repo y los aplica automáticamente al cluster k3s.

---

## 🐳 Build local (opcional)

```bash
docker build -t civil-engineer-web:dev .
docker run -p 8080:80 civil-engineer-web:dev
# → http://localhost:8080
```

---

## ☸️ Manifiestos Kubernetes

| Recurso | Descripción |
|---|---|
| `Namespace` | `civil-web` — aísla todos los recursos |
| `Deployment` | 2 réplicas, rolling update sin downtime |
| `Service` | ClusterIP en puerto 80 |
| `Ingress` | Traefik (default en k3s), HTTPS redirect |
| `Middleware` | Traefik CRD para redirect HTTP→HTTPS |

### TLS con cert-manager (opcional)

```bash
# Instalar cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml

# Crear ClusterIssuer para Let's Encrypt
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: tu@email.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: traefik
EOF
```

Luego descomentá el bloque `tls` en `k8s/ingress.yaml`.

---

## 🔄 Flujo GitOps completo

```
git push → GitHub Actions → build + push imagen → commit tag actualizado
                                                         ↓
                                               ArgoCD detecta cambio
                                                         ↓
                                              kubectl apply en k3s
                                                         ↓
                                           Rolling update sin downtime ✅
```

---

## 🎨 Personalización del sitio

Editá `frontend/index.html` para cambiar:
- **Nombre del estudio** → buscar `CR / Ingeniería` y `Construcciones Rodríguez`
- **Estadísticas del hero** → sección `.hero-stats`
- **Servicios** → `.services-grid`
- **Proyectos** → `.projects-grid` (reemplazar imágenes de Unsplash por fotos reales)
- **Datos de contacto** → sección `#contacto`
- **Colores** → variables CSS en `:root`
