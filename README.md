# Deploy .NET API to Railway App
<hr>

# O que é o Railway App

[Railway.app](http://Railway.app)

Railway app é uma plataforma na nuvem para construção, envio e monitoramento de aplicativos. Plataformas como essa geralmente visam simplificar o processo de desenvolvimento, fornecendo ferramentas e automações para facilitar a construção e gerenciamentos de aplicativos. 

No Railway é possivel você implementar aplicações de diversas linguagens, banco de dados, volumes e implementar integração CI/CD via github. Na criação de sua primeira conta você recebe $5 para gastar como quiser na plataforma, além dos planos de upgrades terem um ótimo custo benefício também.

# Implementar uma Web Api .NET no Railway APP

Nosso objetivo é implementar uma WebApi .NET no Railway App, utilizando o mesmo método podemos implementar projetos MVC e talvez outros tipos de projetos(não fiz outros testes, fica em aberto para quem tiver inteiresse). Para realizar o deploy da nossa WebApi vamos utilizar o docker, ele é essencial nesse processo, caso você não tenha instalado irei deixar link de um tutorial de como baixar, instalar e configura-lo.

# Tutoriais

- [Como Instalar o .NET](https://balta.io/blog/dotnet-instalacao-configuracao-e-primeiros-passos)
- [Como instalar o Visual Studio Code](https://balta.io/blog/visual-studio-code-instalacao-customizacao)
- [Como instalar o Visual Studio 2022](https://learn.microsoft.com/pt-br/visualstudio/install/install-visual-studio?view=vs-2022)
- [Como instalar e configurar o Docker](https://balta.io/blog/docker-instalacao-configuracao-e-primeiros-passos)

# Como configurar o projeto utilizando Visual Studio Code - CLI

## Iniciar um projeto WebApi .NET C#

- Cria uma pasta para seu projeto e dentro dela abra o Visual Studio Code e o prompt, execute para criar o projeto:
    
    ```bash
    dotnet new webapi -o WebApiDeployExample
    ```
    
- Execute para criar a solução:
    
    ```bash
    dotnet new sln
    ```
    
- Execute para vincular o projeto a solução
    
    ```bash
    dotnet sln add WebApiDeployExample
    ```
    

## Configurando o Docker no projeto

- Crie um arquivo ‘.dockerfile’ na raiz da solução(Mesma localização do seu arquivo .sln), preencha com os seguintes itens:
    
    ```yaml
    **/.classpath
    **/.dockerignore
    **/.env
    **/.git
    **/.gitignore
    **/.project
    **/.settings
    **/.toolstarget
    **/.vs
    **/.vscode
    **/*.*proj.user
    **/*.dbmdl
    **/*.jfm
    **/azds.yaml
    **/bin
    **/charts
    **/docker-compose*
    **/Dockerfile*
    **/node_modules
    **/npm-debug.log
    **/obj
    **/secrets.dev.yaml
    **/values.dev.yaml
    LICENSE
    README.md
    !**/.gitignore
    !.git/HEAD
    !.git/config
    !.git/packed-refs
    !.git/refs/heads/**
    ```
    

- Crie um arquivo ‘Dockerfile’ dentro do projeto(Mesma localização do seu arquivo .csproj), preencha com o seguinte template:
    
    ```yaml
    #See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.
    
    FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
    USER app
    WORKDIR /app
    EXPOSE 8080
    EXPOSE 8081
    
    FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
    ARG BUILD_CONFIGURATION=Release
    WORKDIR /src
    COPY ["[NomeDoProjeto]/[NomeDoProjeto].csproj", "[PastaDoProjeto]/"]
    RUN dotnet restore "./[NomeDoProjeto]/[NomeDoProjeto].csproj"
    COPY . .
    WORKDIR "/src/[NomeDoProjeto]"
    RUN dotnet build "./[NomeDoProjeto].csproj" -c $BUILD_CONFIGURATION -o /app/build
    
    FROM build AS publish
    ARG BUILD_CONFIGURATION=Release
    RUN dotnet publish "./[NomeDoProjeto].csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false
    
    FROM base AS final
    WORKDIR /app
    COPY --from=publish /app/publish .
    ENTRYPOINT ["dotnet", "[NomeDoProjeto].dll"]
    ```
    

## Configurando Railway no projeto

- Crie um arquivo ‘railway.toml’ na raiz da solução(Mesma localização do seu arquivo .sln), preencha com os seguintes itens:
    
    ```toml
    [build]
    builder = "dockerfile"
    dockerfilePath = "./WebApiDeployExample/Dockerfile"
    
    [deploy]
    startCommand = "dotnet WebApiDeployExample.dll"
    restartPolicyType = "never"
    ```
    
- No projeto em Program.cs adicione o seguinte código antes de ‘var app = builder.Build();’:
    
    ```csharp
    var port = Environment.GetEnvironmentVariable("PORT") ?? "8081";
    builder.WebHost.UseUrls($"http://*:{port}");
    ```
    
- Comente ‘app.UseHttpRedirection();’
- Caso queira visualizar o Swagger ao publicar retire ‘app.UseSwagger();’ e ‘app.UseSwaggerUI();’ de dentro das condições de ambiente de desenvolvimento.
    - Antes
        
        ```csharp
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }
        ```
        
    - Depois
        
        ```csharp
        app.UseSwagger();
        app.UseSwaggerUI();
        ```
        

# Como configurar o projeto utilizando Visual Studio 2022 - Interface

## Iniciar um projeto WebApi .NET C#

- Procure por Web API
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/0e7c8450-b3eb-4488-a789-92e5a1342d6e)

    
- Configure Nome do projeto, Local e Nome da solução
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/84bc0341-2af1-4f3f-b56e-b9c10a72a3cf)

    

- Selecione versão do .NET, habilite o Docker e selecione Linux como sistema operacional do Docker
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/7abf90d9-55a1-4996-bb38-4c3f78af532c)

    

## Configurando o Docker no projeto

- Via interface os arquivos Docker são configurados automaticamente.

## Configurando Railway no projeto

- Crie um arquivo ‘railway.toml’ na raiz da solução(Mesma localização do seu arquivo .sln), preencha com os seguintes itens:
    
    ```toml
    [build]
    builder = "dockerfile"
    dockerfilePath = "./WebApiDeployExample/Dockerfile"
    
    [deploy]
    startCommand = "dotnet WebApiDeployExample.dll"
    restartPolicyType = "never"
    ```
    
- No projeto em Program.cs adicione o seguinte código antes de ‘var app = builder.Build();’:
    
    ```csharp
    var port = Environment.GetEnvironmentVariable("PORT") ?? "8081";
    builder.WebHost.UseUrls($"http://*:{port}");
    ```
    
- Comente ‘app.UseHttpRedirection();’
- Caso queira visualizar o Swagger ao publicar retire ‘app.UseSwagger();’ e ‘app.UseSwaggerUI();’ de dentro das condições de ambiente de desenvolvimento.
    - Antes
        
        ```csharp
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }
        ```
        
    - Depois
        
        ```csharp
        app.UseSwagger();
        app.UseSwaggerUI();
        ```
        

# Como subir o projeto para o github e fazer deploy no RailwayApp

## Subindo o projeto para o github

- Adicione um arquivo ‘.gitignore’ na raiz da solução(Mesma localização do seu arquivo .sln), preencha com os seguinte ou itens a critério:
    
    ```yaml
    .vs
    ```
    
- Crie um repositório local na raiz da solução utilizando
    
    ```bash
    git init
    ```
    
- Crie um repositório vázio no github e conecte ao seu repositório local utilizando os comandos
    
    ```bash
    git remote add origin https://github.com/lucasdwn/WebApiDeployExample.git
    git branch -M main
    git add .
    git commit -m 'First Commit'
    git push -u origin main
    ```
    

## Fazendo deploy com o Railway

- No dashboard do railway após ter logado com o Github, crie um novo projeto e selecione Deploy from GitHub repo
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/eb9b0251-7911-4e1a-a2de-db96c92c4a13)

    

- Busque pelo seu repositório e selecione 'Deploy Now’
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/f818d0ba-deae-44a1-8e02-bed4ae0b5f37)

    

- Após finalizar o deploy adicione um domínio
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/36ab2c51-4535-41ea-af12-0226b20c2b76)

    

- Acesse o Swagger da WebApi caso tenha removido o uso do mesmo apenas de desenvolvimento ( url/swagger/index.html)
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/6a734a2a-39eb-435d-959c-a44dd02b3cbe)

    
- Utilize o Insomnia ou Postman para enviar uma requisição para a API caso não tenha removido  Swagger de desenvolvimento.
    
    ![image](https://github.com/lucasdwn/WebApiDeployExample/assets/68930336/38903aaa-b77a-46a6-b56a-d1e6a05707cd)

    

# Repositório de exemplo

- [Repositório](https://github.com/lucasdwn/WebApiDeployExample)

# Documentações e referências

- [Documentação RailwayApp](https://docs.railway.app/)
- [Referência Deploy WebApi .NET no RailwayApp](https://www.iliabedian.com/blog/deploy-dotnet-app-on-railway-with-docker)

# Contatos

- [LinkedIn](https://www.linkedin.com/in/lucascostadwn/)
- [Github](https://github.com/lucasdwn)
- [E-mail](mailto:lfcosta0804@gmail.com)
