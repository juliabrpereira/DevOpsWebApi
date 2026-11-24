# Laboratório Docker

API ASP.NET Core (.NET 10) executada em containers, com staging, production, health check e rollback.

No Linux/macOS, use `curl` em vez de `curl.exe`.

## Construção

```powershell
docker build -t devops-webapi:1.0.0 .
```

## Execução

```powershell
docker run -d --name devops-api -p 8080:8080 devops-webapi:1.0.0
```

Se a porta 8080 do host estiver ocupada, use outra, por exemplo `-p 8083:8080`.

## Staging

```powershell
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

## Production

```powershell
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.0.0 devops-webapi:1.0.0
```

## Health Check

```powershell
curl.exe -f http://localhost:8081/health
curl.exe -f http://localhost:8082/health
```

## Nova versão

A versão 1.1.0 acrescenta o endpoint `/version`. Ela vai primeiro para staging:

```powershell
docker build -t devops-webapi:1.1.0 .
docker rm -f devops-api-staging
docker run -d --name devops-api-staging -p 8081:8080 -e APP_ENVIRONMENT=staging -e APP_VERSION=1.1.0 devops-webapi:1.1.0
curl.exe -f http://localhost:8081/health
```

Após a validação, é promovida para production:

```powershell
docker rm -f devops-api-prod
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.1.0 devops-webapi:1.1.0
```

## Falha controlada

A imagem `devops-webapi:1.2.0` foi construída com uma alteração temporária em `/health`, que passou a responder HTTP 500:

```csharp
app.MapGet("/health", () =>
    Results.StatusCode(
        StatusCodes.Status500InternalServerError
    )
);
```

O código atual já contém o endpoint corrigido.

```powershell
docker rm -f devops-api-prod
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.2.0 devops-webapi:1.2.0
curl.exe -f http://localhost:8082/health
```

Resultado: `curl: (22) The requested URL returned error: 500`

## Rollback

```powershell
docker rm -f devops-api-prod
docker run -d --name devops-api-prod -p 8082:8080 -e APP_ENVIRONMENT=production -e APP_VERSION=1.1.0 devops-webapi:1.1.0
curl.exe -f http://localhost:8082/health
```

## Docker Compose

Remova antes os containers criados manualmente, pois os nomes são os mesmos:

```powershell
docker rm -f devops-api-staging devops-api-prod
docker compose up -d
docker compose ps
docker compose down
```
