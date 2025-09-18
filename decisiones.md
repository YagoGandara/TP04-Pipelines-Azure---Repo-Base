# Decisiones - TP04 Azure DevOps Pipelines (SelfHosted)

## 1) Stack elegido y Estructura del Repo

- **Frontend**: React + Vite + TypeScript
  **Motivo**: Build rápido, salida estática clara (`front/dist`)

- **Backend**: .NET 8 WebAPI
  **Motivo**: Es el que mas hemos usado, y sus comandos son sencillos (`restore/build/publish`)

- **Monorepo**:
    /front → Vite + React (TS)
    /back → .NET 8 WebAPI (back.csproj)
    /azure-pipelines.yml (raíz)

---

## 2) Diseño del Pipeline (Stages, jobs y artefactos)

- **Fuente**: Github (trigger desde la rama `main`)
- **Pool/Agente**: `SelfHosted` (agente local "Agent-Local")
- **Stage**: `CI` con **2 Jobs**
- **Build_Front**:
    a) Cache `node_modules` (`Cache@2` con clave basada en el `package-lock.json`)
    b) `npm install`/`npm ci` + `npm run build` (Vite)
    c) **Artefacto**: `front-dist` (`front/dist`)
- **Build_Back**:
    a) Cache de NuGet (`Cache@2` con clave sobre `/x.csproj`)
    b) `dotnet restore` -> `dotnet build -c Release` -> `dotnet publish -c Release`
    c) **Artefacto**: `back-publish` (`back/bin/Release/publish`)
- **Variables Clave**:
    - `frontDir = front`
    - `backProj=back/back.csproj`
    - `frontOut=$(Build.SourcesDirectory)/front/dist`
    - `backPublish=$(Build.SourcesDirectory)/back/bin/Release/publish`

---

## 3) Fundamento de Decisiones 
- **Self-Hosted**: lo pide el TP, y su control de versiones esta alineada con Node/.NET
- **Vite y React (TS)**: lo conozco un poco mas, sus builds son rápidas y la configuración es poca (y el artefacto estático es simple) 
- **.NET 8**: puedo usar el Swagger, que es lo que me traia problemas para probar como se veia 
- **Jobs Separados**: si fallaba estaba mas aislados, sus logs son mas claros y las caches independientes

---

## 4) Paso a Paso de todo

1. **Estructura**: se creo `/front`y `/back`
2. **Front**
    - `npm create vite@latest front -- --template react-ts`
    - `cd front && npm install`
    - Hubo error de `tsc` -> `npm i -D typescript vite` → validación `npx tsc -v (5.8.3)`
    - `npm run build` dio bien -> `front/dist` generado
3. **Back**
    - `dotnet new webapi -o back`
    - `cd back && dotnet restore && dotnet build -c Release`
    - `dotnet run` -> no andaba 
    -  `dotnet dev-certs https --trust` + variables para `https://localhost:7298`
    - `dotnet run` -> Ahora anduvo
4. **Repositorio**:
    - `git add . && git commit -m "TP04: monorepo front + back + pipeline"` -> el pipeline tenía un error en algún lugar de comillas que me comí
    - `git push -u origin main` (GitHub).
5. **Pipeline**:
    - YAML en **raíz** (`azure-pipelines.yml`), pool `SelfHosted`
    - Runs exitosos, publicación de artefactos

--- 

## 6) Problemas y Soluciones:

- **`npm ERR! enoent package.json` en `/front`**
    Ocurrió porque la carpeta estaba vacía, por lo que inicialicé con Vite y reinstale las dependencias

- **`"tsc" no se reconoce`**  
    Faltaba el TypeScript en dev deps, lo solucioné ejecutando
    -  `npm install` + `npm i -D typescript vite` y controlé con `npx tsc -v`

- **Warning `UseHttpsRedirection`**
    El puerto HTTPS no estaba definido en dev, ejecuté:
    -  `dotnet dev-certs https --trust` + variables

- **YAML “quoted scalar/end of stream”**
    En algún lugar había comillas sin cerrar, le pedí al chat que lo limpie, y funcionó
