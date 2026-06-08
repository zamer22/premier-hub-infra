# Premier-hub-infra

Manifiestos de Kubernetes para Premier Hub. Se aplican directamente en el servidor — no hay pipeline de deploy automatizado para este repo.

## Servicios

| Servicio | Imagen | Descripción |
|----------|--------|-------------|
| `pagina` | `premier-pagina` | Frontend (Nginx, puerto 80) |
| `api` | `premier-api` | Backend Node.js (puerto 4000) |
| `ml` | `premier-ml` | Servicio de predicciones ML (puerto 8080) |

## Ambientes

### prod
- **Frontend:** https://app.zamer-o.com
- **Namespace:** `prod`
- **Imágenes:** tag `latest`

| Servicio | Tipo | Puerto externo (NodePort) |
|----------|------|--------------------------|
| pagina | NodePort | 30300 |
| api | NodePort | 30400 |
| ml | ClusterIP | — (interno: `10.43.53.230:8080`) |

### preprod
- **Frontend:** https://app-preprod.zamer-o.com
- **Namespace:** `preprod`
- **Imágenes:** tag `preprod`

| Servicio | Tipo | Puerto externo (NodePort) |
|----------|------|--------------------------|
| pagina | NodePort | 30301 |
| api | NodePort | 30401 |
| ml | ClusterIP | — (interno: `ml.preprod.svc.cluster.local:8080`) |

## Estructura

```
k8s/
├── prod/
│   ├── api-deployment.yaml
│   ├── api-service.yaml
│   ├── ml-deployment.yaml
│   ├── ml-service.yaml
│   ├── pagina-deployment.yaml
│   └── pagina-service.yaml
└── preprod/
    ├── api-deployment.yaml
    ├── api-service.yaml
    ├── ml-deployment.yaml
    ├── ml-service.yaml
    ├── pagina-deployment.yaml
    └── pagina-service.yaml

versions.env          # Tags de imagen actualmente desplegados por ambiente
```

## Aplicar manifiestos

Conectarse al servidor y ejecutar:

```bash
# Producción
kubectl apply -f k8s/prod/

# Preprod
kubectl apply -f k8s/preprod/
```

## Secretos

El API requiere un Secret llamado `api-secrets` en cada namespace. Los valores de cada variable se encuentran en el documento **`06_Configuracion_de_seguridad.docx`**:

```bash
kubectl create secret generic api-secrets \
  --namespace=prod \
  --from-literal=SUPABASE_URL=<ver 06_Configuracion_de_seguridad.docx> \
  --from-literal=SUPABASE_SERVICE_KEY=<ver 06_Configuracion_de_seguridad.docx> \
  --from-literal=APIFOOTBALL_KEY=<ver 06_Configuracion_de_seguridad.docx> \
  --from-literal=NEWS_API_KEY=<ver 06_Configuracion_de_seguridad.docx> \
  --from-literal=COOKIE_SECRET=<ver 06_Configuracion_de_seguridad.docx>
```

## Notas

- `imagePullPolicy: Never` — las imágenes deben existir localmente en el nodo antes de aplicar.
- Todos los pods usan `dnsPolicy: None` con nameservers externos (8.8.8.8, 1.1.1.1). Por eso el servicio `ml` en prod usa ClusterIP fija (`10.43.53.230`) en vez de nombre DNS interno.
- `versions.env` registra los tags actualmente desplegados, actualizado por los workflows de CI/CD de los otros repos.
