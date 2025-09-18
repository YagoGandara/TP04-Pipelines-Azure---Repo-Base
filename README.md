# TP04 — Azure DevOps Pipelines (Self-Hosted) · Monorepo Front + Back

Mono-repo con **frontend** (React + Vite + TypeScript) y **backend** (.NET 8 WebAPI) con **pipeline CI en YAML** que compila ambos y publica artefactos, ejecutándose en **agente Self-Hosted**.

---

## Estructura del Repo
├─ front/ # Vite + React (TS)
│ ├─ package.json
│ └─ src/ ...
├─ back/ # .NET 8 WebAPI
│ ├─ back.csproj # (tu .csproj puede llamarse distinto)
│ └─ Program.cs
├─ azure-pipelines.yml # pipeline CI (en raíz)
├─ README.md
└─ decisiones.md


---

## Prerrequisitos (máqina/agentes usados)

- **Node.js**
- **npm**
- **.NET SDK**
- **Git**

---

## Correr Localmente

# Front (Vite + React + TS)
```bash
cd front
npm install #la 1ra vez, si el package-lock.json está, podes usar npm ci
npm run dev #http://localhost:5173
npm run build #genera front/dist

```

# Back (.NET 8 WebAPI)
```bash
cd back
dotnet restore
dotnet build -c Release
dotnet run # http://localhost:5298/swagger

```

# HTTPS en dev (lo tuve que hacer porque no andaba)

```bash
dotnet dev-certs https --trust
$env:ASPNETCORE_URLS = "http://localhost:5298;https://localhost:7298"
$env:ASPNETCORE_HTTPS_PORT = "7298"
dotnet run            # HTTPS: https://localhost:7298  (Swagger: /swagger)

```

---

##Pipeline CI (Azure DevOps)

- Fuente: **GitHub** (push main -> dispara CI)
- Pool: SelfHosted (agente local)
- Stage: CI con 2 jobs:
    1) Build_Front: `npm install/ci` + `npm run build` -> artifact `front-dist`
    2)Build_Back: `dotnet restore` + `build` + `publish` -> artifact `back-publish`

- Variables relevantes de YAML:
    1) frontDir: front
    2) backProj: back/back.csproj
    3) frontOut: $(Build.SourcesDirectory)/front/dist
    4) backPublish: $(Build.SourcesDirectory)/back/bin/Release/publish

