# 2gn_frontend

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

## Cloud Build deploy (Google Cloud Run)

Run these commands from the workspace root (`2GN_WEBSITE`):

```bash
# deploy backend Cloud Run service
gcloud builds submit --config=2gn_backend/cloudbuild.yaml 2gn_backend

# deploy frontend Cloud Run service
gcloud builds submit --config=2gn_frontend/cloudbuild.yaml 2gn_frontend
```

Before deploying, verify substitutions in both `cloudbuild.yaml` files:
- backend: `2gn_backend/cloudbuild.yaml`
- frontend: `2gn_frontend/cloudbuild.yaml`

Important values to set:
- `_REGION`
- `_SERVICE_NAME`
- backend `_CORS_ORIGIN`
- frontend `_VUE_APP_API_BASE`

## Commit policy

Keep these files committed to git:
- `Dockerfile`
- `cloudbuild.yaml`
- `nginx.conf`
- `.env.example` / `.env.*.example`

Do NOT commit secrets or local environment files:
- `.env`
- `.env.production`
- any file containing real credentials, keys, or tokens

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
